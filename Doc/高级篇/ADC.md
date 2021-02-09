## ADC

模拟数字转换器即A/D转换器，或简称ADC，通常是指一个将模拟信号转变为数字信号的电子元件。通常的模数转换器是将一个输入电压信号转换为一个输出的数字信号。

![ADC.jpeg](https://github.com/zihan987/Raspberry-Pi-Pico/blob/main/images/ADC.jpeg?raw=true)

### MicroPython

```python
import machine
import utime

sensor_temp = machine.ADC(4)
conversion_factor = 3.3 / (65535)

while True:
    reading = sensor_temp.read_u16() * conversion_factor
    
    # The temperature sensor measures the Vbe voltage of a biased bipolar diode, connected to the fifth ADC channel
    # Typically, Vbe = 0.706V at 27 degrees C, with a slope of -1.721mV (0.001721) per degree. 
    temperature = 27 - (reading - 0.706)/0.001721
    print(temperature)
    utime.sleep(2)
```

### C/C++

```c
#include <stdio.h>
#include "pico/stdlib.h"
#include "hardware/adc.h"  // For ADC input:
#include "hardware/dma.h"

#include "pico/multicore.h"// For resistor DAC output:
#include "hardware/pio.h"
#include "resistor_dac.pio.h"

// Channel 0 is GPIO26
#define CAPTURE_CHANNEL 0
#define CAPTURE_DEPTH 1000

uint8_t capture_buf[CAPTURE_DEPTH];

void core1_main();

int main() {
    stdio_init_all();

    // Send core 1 off to start driving the "DAC" whilst we configure the ADC.
    multicore_launch_core1(core1_main);

    // Init GPIO for analogue use: hi-Z, no pulls, disable digital input buffer.
    adc_gpio_init(26 + CAPTURE_CHANNEL);

    adc_init();
    adc_select_input(CAPTURE_CHANNEL);
    adc_fifo_setup(
        true,    // Write each completed conversion to the sample FIFO
        true,    // Enable DMA data request (DREQ)
        1,       // DREQ (and IRQ) asserted when at least 1 sample present
        false,   // We won't see the ERR bit because of 8 bit reads; disable.
        true     // Shift each sample to 8 bits when pushing to FIFO
    );

    adc_set_clkdiv(0);

    printf("Arming DMA\n");
    sleep_ms(1000);
    // Set up the DMA to start transferring data as soon as it appears in FIFO
    uint dma_chan = dma_claim_unused_channel(true);
    dma_channel_config cfg = dma_channel_get_default_config(dma_chan);

    // Reading from constant address, writing to incrementing byte addresses
    channel_config_set_transfer_data_size(&cfg, DMA_SIZE_8);
    channel_config_set_read_increment(&cfg, false);
    channel_config_set_write_increment(&cfg, true);

    // Pace transfers based on availability of ADC samples
    channel_config_set_dreq(&cfg, DREQ_ADC);

    dma_channel_configure(dma_chan, &cfg,
        capture_buf,    // dst
        &adc_hw->fifo,  // src
        CAPTURE_DEPTH,  // transfer count
        true            // start immediately
    );

    printf("Starting capture\n");
    adc_run(true);

    // Once DMA finishes, stop any new conversions from starting, and clean up
    // the FIFO in case the ADC was still mid-conversion.
    dma_channel_wait_for_finish_blocking(dma_chan);
    printf("Capture finished\n");
    adc_run(false);
    adc_fifo_drain();

    // Print samples to stdout so you can display them in pyplot, excel, matlab
    for (int i = 0; i < CAPTURE_DEPTH; ++i) {
        printf("%-3d, ", capture_buf[i]);
        if (i % 10 == 9)
            printf("\n");
    }
}

#define OUTPUT_FREQ_KHZ 5
#define SAMPLE_WIDTH 5
// This is the green channel on the VGA board
#define DAC_PIN_BASE 6

void core1_main() {
    PIO pio = pio0;
    uint sm = pio_claim_unused_sm(pio0, true);
    uint offset = pio_add_program(pio0, &resistor_dac_5bit_program);
    resistor_dac_5bit_program_init(pio0, sm, offset,
        OUTPUT_FREQ_KHZ * 1000 * 2 * (1 << SAMPLE_WIDTH), DAC_PIN_BASE);
    while (true) {
        // Triangle wave
        for (int i = 0; i < (1 << SAMPLE_WIDTH); ++i)
            pio_sm_put_blocking(pio, sm, i);
        for (int i = 0; i < (1 << SAMPLE_WIDTH); ++i)
            pio_sm_put_blocking(pio, sm, (1 << SAMPLE_WIDTH) - 1 - i);
    }
}
```

CMakeList.txt:

```cmake
add_executable(adc_dma_capture
        dma_capture.c
        )

pico_generate_pio_header(adc_dma_capture ${CMAKE_CURRENT_LIST_DIR}/resistor_dac.pio)

target_link_libraries(adc_dma_capture
		pico_stdlib
		hardware_adc
		hardware_dma
		# For the dummy output:
		hardware_pio
		pico_multicore
		)

# create map/bin/hex file etc.
pico_add_extra_outputs(adc_dma_capture)

# add url via pico_set_program_url
example_auto_set_url(adc_dma_capture)
```