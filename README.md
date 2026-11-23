# ECE-528-HW-3
Submission for HW 3 for ECE 528

1.
* UCAxBRW (Baud Rate Control Word): Sets the baud rate when a value of N is written to it (N = Clock Frequency / Baud Rate)
* UCAxMCTLW (Modulation Control Word): 
* UCAxCTLW0 (Control Word 0): 
* UCAxRXBUF (Receive Buffer):  Contains the last received character from the receive shift register
* UCAxTXBUF (Transmit Buffer): Holds the data waiting to be moved into the transmit shift register and transmitted on UCAxTXD.

2. UCAxSTATW, the status register of the eUSCI_A module, lets software check what the UART is doing and whether any receive errors have occurred. UCBUSY (bit 0) is set to 1 while a transmit or receive operation is active and is 0 when the module is idle. UCFE (bit 6) indicates if a framing error has occurred. UCPE (bit 4) indicates if a parity error has occurred. UCOE (bit 5) indicates if an overrun error has occurred.

3. 
clock frequency = 12 MHz = 12000000 Hz
baud rate = 115200
N = 12000000/115200 = 104.1667

Since N ≥ 16, we use the oversampling mode (UCOS16 = 1)
UCBRx = INT(N / 16) = INT(104.1667 / 16) = INT(6.5104) = 6

6 should be written to the UCAxBRW register.

4.
a.
````c
void UART2_Init (void) {

    // Hold reset while configuring
    EUSCI_A2->CTLW0 |= 0x0001;

    // data length = 8
    // parity = none
    // stop bits = 1
    // MSB first
    // clock source: SMCLK
    EUSCI_A2->CTLW0 |= 0x2880;

    // baud rate = 115200
    EUSCI_A2->BRW = 6;

    // Configure P3.2 and P3.3 for primary module function
    P3->SEL0 |=  0x0C;
    P3->SEL1 &= ~0x0C;

    // Release reset
    EUSCI_A2->CTLW0 &= ~0x0001;

}
````

b.

5. The function of the eUSCI_B module. In terms of synchronization, SPI is synchronous while UART is asynchronous. In terms of clocking, SPI requires a dedicated clock line while UART does not. In terms of signaling, SPI uses 3-4 lines while UART only uses two (TX and RX). In terms of data framing, data in SPI is framed by the clock cycles, while data in UART is framed with a start bit, data bits, optional parity, and a stop bit.

6.
a.
* SCLK (Serial Clock): Provides the clock signal that synchronizes data transfer between master and slave
* CS (Chip Select): Used by master to select which slave device it wants to communicate with.
* MISO (Master In Slave Out): Carries data from slave to master.
* MOSI (Master Out Slave In): Carries data from master to slave.

b. If multiple SPI peripheral devices have their CS line asserted simultaneously on the MSP432, they will fight over the voltage, possibly leading to a short circuit. 

7.
a. The UCBxCTLW0 (Control Word 0) register configures the operational mode, addressing, clock source, and basic transmit/receive behavior of the eUSCI_Bx interface.

b. UCSYNC (bit 8) enables either the synchronous or asynchronous mode. UCMODEx (bits 10:9) select the eUSCI mode. MCMST (bit 11) selects either master mode or slave mode. UCMSB (bit 13) determines if the MSB or LSB is received/transmitted first. 

c. The UCSSELx field (bits 7:6) selects the clock source used by the eUSCI_B module. Value 00b uses the UCxCLK in slave mode only. Value 01b uses the ACLK in master mode only. Values 10b and 11b use the SMCLK in master mode only.

8.
a.
````c
void EUSCI_B0_SPI_Init (void) {
    // Hold eUSCI_B0 in reset while configuring
    EUSCI_B0->CTLW0 |= 0x0001;

    // SPI Mode: 0
    // Data Length: 8 bits
    // Bit Order: Most Significant Bit (MSB) First
    // Clock Source: SMCLK (12 MHz)
    // SPI format: 3-wire SPI
    EUSCI_B0->CTLW0 |= 0xA880;

    // SPI clock = 12 MHz / 12 = 1 MHz
    EUSCI_B0->BRW = 12;

    // Configure pins
    P1->SEL0 |=  0xE0;
    P1->SEL1 &= ~0xE0;

    P9->SEL0 &= ~0x01;
    P9->SEL1 &= ~0x01;
    P9->OUT  |=  0x01;
    P9->DIR  |=  0x01;

    // Release reset
    EUSCI_B0->CTLW0 &= ~0x0001;
}
````
b.

## References

https://www.ti.com/lit/pdf/SLAU424 (5)

https://www.tristondoesprojects.com/meet-the-msp430-usci-module (5)

https://nodeloop.org/guides/spi-interface-guide/ (6)

https://electronics.stackexchange.com/questions/512702/spi-mutiple-slave-configuration-miso-line-conflict (6)

https://www.ti.com/lit/ug/slau424f/slau424f.pdf?ts=1795128214222&ref_url=https%253A%252F%252Fwww.bing.com%252F (1)(2)(7)