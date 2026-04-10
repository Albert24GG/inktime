# InkTime Smartwatch

## Block Diagram

![System Block Diagram](./block_diagram.svg)

## Hardware Functionality

### Power Management
The power path starts at the **USB-C connector (5V)**, which feeds the **BQ25180 LiPo Charger**. This IC handles the 250mAh battery and provides a system rail (VSYS). To ensure a stable 3.3V regardless of the battery's state of charge, an **RT6160 Buck-Boost regulator** was used. This is critical because it keeps the MCU and E-paper display running even as the battery voltage drops. Furthermore, **MAX17048 Fuel Gauge** was included to get the battery percentage over I2C.

### Compute & Connectivity
The MCU chosen is **nRF52840**. It manages:
*   **I2C Bus:** Shared by the charger, regulator, fuel gauge, IMU, and haptic driver.
*   **SPI Bus:** Used exclusively for the E-paper display to handle fast screen refreshes.
*   **USB:** For firmware recovery and battery charging.
*   **RF:** A 2.4GHz trace leads to a chip antenna for BLE communication with a smartphone.

### Peripherals
*   **Display:** A 1.54" 200x200 E-paper module. Since these displays can leak power even when "off," we added a PFET-based power gate to completely cut VCC to the display during deep sleep.
*   **IMU:** The **BMA421** accelerometer handles step counting in hardware, waking the MCU only when a step is detected to save power.
*   **Haptics:** A **DRV2605L** driver controls an ERM shaker motor for haptic feedback.

## MCU Pinout Mapping

| Peripheral | Pin(s) | Function | Justification |
| :--- | :--- | :--- | :--- |
| **I2C SDA** | P0.06 | Data | Shared bus for sensors and power ICs. |
| **I2C SCL** | P0.07 | Clock | Shared bus for sensors and power ICs. |
| **SPI SCK** | P0.02 | Clock | Dedicated high-speed SPI for E-Paper. |
| **SPI MOSI** | P0.03 | Data Out | Dedicated high-speed SPI for E-Paper. |
| **SPI CS** | P0.05 | Chip Sel | Dedicated high-speed SPI for E-Paper. |
| **EPD DC** | P0.15 | Data/Cmd | Required for display command logic. |
| **EPD RST** | P0.16 | Reset | Hardware reset for display controller. |
| **EPD BUSY** | P0.17 | Input | MCU checks this before display updates. |
| **EPD GATE** | P1.01 | PFET Ctrl | Cuts display power. |
| **USB D+ / D-** | D+ / D- | USB Data | Differential pair for firmware/recovery. |
| **VBUS** | VBUS | Detect | Used by MCU to detect USB cable attachment. |
| **ANT1** | ANT | RF Out | 50-ohm matched trace to 2.4GHz chip antenna. |
| **X1** | XC1 / XC2 | 32MHz | High-speed clock for Radio/System. |
| **X2** | P0.00 / P0.01 | 32.768kHz | Low-power clock for RTC and Sleep timing. |
| **IMU INT1** | P0.08 | Wake-up | Primary IMU interrupt. |
| **IMU INT2** | P1.08 | Gesture | Secondary IMU interrupt for advanced gestures/events. |
| **PMIC INT** | P0.11 | Charger | Signals charging status changes. |
| **Fuel Alert** | P0.10 | Low Batt | Triggers when battery hits a critical % threshold. |
| **Haptic EN** | P0.12 | Enable | Power control for the haptic driver. |
| **Button Up** | P0.13 | Input | Navigation (Active Low). |
| **Button Dn** | P1.02 | Input | Navigation (Active Low). |
| **Button Ent** | P0.14 | Input | Enter / Confirm (Active Low). |
| **SWD Bus** | SWDIO/CLK | Debug | Programming via Tag-Connect TC2030. |

## Design Log

During the development of the PCB and mechanical integration, several adjustments were made to ensure reliability and manufacturability:

*   **EPD Capacitor Tweak:** Changed **EPD_C5 to 1uF** (previously 100nF) after double-checking the display driver datasheet requirements for the internal charge pump.
*   **Silkscreen Optimization:** Because JLCPCB has a minimum text height of 1mm and our board is very dense, I removed labels for small passives. Only major components, connectors, and test pads have silkscreen labels to avoid a "messy" print.
*   **Bottom Layer Connectors:** The pads for the **Battery** and the **Shaker Motor** were moved to the **Bottom Layer**. Since these components live in the bottom of the case (under the PCB), this makes hand-soldering them much easier during assembly.
*   **Inner Layer Pad Cleanup:** To prevent manufacturing errors (shorts) with via-in-pad routing on the dense MCU footprint, I used the `Remove All Non-Functional Pads` option for the internal layers.
*   **Component Sizing:** While I aimed for 0201/0402 where possible, I used **0603 footprints** for the large inductors and decoupling caps in the DC/DC section to meet the voltage/current ratings.
*   **RF Optimization:** The antenna feedline (RF trace) was designed with a width of **0.1505mm**. This was calculated for a **50-ohm impedance match** based on the 4-layer stackup (Coplanar Waveguide with Ground).
*   **Accessibility:** All test pads were placed near the edges of the PCB so they can be reached easily with probes even when the display is mounted.
*   **GND & Thermal Management:** I applied a copper pour on the Top Layer with **via stitching** to the internal Ground Plane.
*   **Antenna Safety:** The antenna keepout zone follows the datasheet dimensions exactly. I also added **via fencing** around the keepout area to minimize interference with the rest of the components.

## Bill of Materials (BOM)

| Qty | Value | Parts | Device | Package | Description | Product Link | Datasheet Link |
|---|---|---|---|---|---|---|---|
| 17 | 1uF/50V | C15, C29, C30, C31, C32, C37, C38, EPD_C1, EPD_C2, EPD_C5, EPD_C6, EPD_C7, EPD_C8, EPD_C9, EPD_C10, EPD_C11, EPD_C12 | GRM155R61H105KE05D | 0402 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/1609005-GRM155R61H105KE05D/C1518208) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2201180430_Murata-Electronics-GRM155R61H105KE05D_C1518208.pdf) |
| 6 | 4.7uF/25V | C6, C14, C20, C21, C43, C2-EP-DR | GRM155C61E475ME15D | 0402 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/6643118-GRM155C61E475ME15D/C5718966) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2308111413_Murata-Electronics-GRM155C61E475ME15D_C5718966.pdf) |
| 2 | 10uF/25V | C1-EP-DR, C39 | GRM188R61E106KA73D | 0603 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/320824-GRM188R61E106KA73D/C344022) | [Datasheet](https://www.lcsc.com/datasheet/C344022.pdf) |
| 1 | 10uF/6.3V | C24 | GRM155R60J106ME44D | 0402 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-GRM155R60J106ME44D/C76991) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2304140030_Murata-Electronics-GRM155R60J106ME44D_C76991.pdf) |
| 2 | 22uF/10V | C25, C33 | GRM188R61A226ME15D | 0603 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-GRM188R61A226ME15D/C84419) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/1810311140_Murata-Electronics-GRM188R61A226ME15D_C84419.pdf) |
| 9 | 100nF/25V | C5, C7, C8, C12, C19, C23, C27, C34, C42 | GRM033R61E104KE14D | 0201 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-GRM033R61E104KE14D/C76939) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/1811081532_Murata-Electronics-GRM033R61E104KE14D_C76939.pdf) |
| 1 | 100pF/50V | C11 | GRM0335C1H101JA01D | 0201 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-GRM0335C1H101JA01D/C76922) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2304140030_Murata-Electronics-GRM0335C1H101JA01D_C76922.pdf) |
| 1 | 47nF/16V | C16 | GRM033R61C473KE84D | 0201 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-GRM033R61C473KE84D/C88917) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2304140030_Murata-Electronics-GRM033R61C473KE84D_C88917.pdf) |
| 4 | 12pF/50V | C1, C2, C17, C18 | GRM0335C1H120FA01D | 0201 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/358666-GRM0335C1H120FA01D/C384960) | [Datasheet](https://www.lcsc.com/datasheet/C384960.pdf) |
| 2 | 1pf/50V | C3, C4 | GRM0335C1H1R0WA01D | 0201 | SMD Capacitor | [Link](https://jlcpcb.com/partdetail/172792-GRM0335C1H1R0WA01D/C161411) | [Datasheet](https://www.lcsc.com/datasheet/C161411.pdf) |
| 1 | 470mΩ | R1_EP_DR | KDV02DR470ET | 0201 | SMD Resistor | [Link](https://jlcpcb.com/partdetail/Ohmite-KDV02DR470ET/C4327454) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2404121342_Ohmite-KDV02DR470ET_C4327454.pdf) |
| 1 | 2.2Ω | R_TYPE_SEL | RC0201FR-072R2L | 0201 | SMD Resistor | [Link](https://jlcpcb.com/partdetail/YAGEO-RC0201FR072R2L/C138154) | [Datasheet](https://www.lcsc.com/datasheet/C138154.pdf) |
| 2 | 3.3KΩ | R17, R18 |  RC0201DR-073K3L| 0201 | SMD Resistor | [Link](https://jlcpcb.com/partdetail/YAGEO-RC0201DR073K3L/C851731) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2206241815_YAGEO-RC0201DR-073K3L_C851731.pdf) |
| 2 |  5.1KΩ | R1_USB, R2_USB | RT0201BRD075K1L | 0201 | SMD Resistor | [Link](https://jlcpcb.com/partdetail/YAGEO-RT0201BRD075K1L/C852257) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2210190401_YAGEO-RT0201BRD075K1L_C852257.pdf) |
| 6 | 10KΩ | R5, R7, R8, R9, R2_EP_DR, R_PWR_EPD | RE0201BRE0710KL | 0201 | SMD Resistor | [Link](https://jlcpcb.com/partdetail/YAGEO-RE0201BRE0710KL/C1882653) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2201280400_YAGEO-RE0201BRE0710KL_C1882653.pdf) |
| 1 | 470nH | L7 | FTC252012SR47MBCA | 2520 | SMD Inductor | [Link](https://www.lcsc.com/product-detail/C5832368.html) | [Datasheet](https://www.lcsc.com/datasheet/C5832368.pdf) |
| 1 | 3.9nH | L1 | LQP03TN3N9B02D | 0201 | SMD Inductor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-LQP03TN3N9B02D/C77115) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2304140030_Murata-Electronics-LQP03TN3N9B02D_C77115.pdf) |
| 1 | 10uH | L2 | MLF1608E100JT000 | 0603 | SMD Inductor | [Link](https://jlcpcb.com/partdetail/TDK-MLF1608E100JT000/C275361) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2304140030_TDK-MLF1608E100JT000_C275361.pdf) |
| 1 | 15nH | L3 | LQG15HS15NG02D | 0402 | SMD Inductor | [Link](https://jlcpcb.com/partdetail/MurataElectronics-LQG15HS15NG02D/C576527) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2308111510_Murata-Electronics-LQG15HS15NG02D_C576527.pdf) |
| 1 | 68uH | L5 | 744043680 | SMD, 4.8x4.8mm | SMD Inductor | [Link](https://jlcpcb.com/partdetail/WurthElektronik-744043680/C2045671) | [Datasheet](https://www.we-online.com/components/products/datasheet/744043680.pdf) |
| 1 | USBLC6-2SC6Y | D3 | USBLC6-2SC6Y | SOT-23-6L | ESD Protection | [Link](https://jlcpcb.com/partdetail/STMicroelectronics-USBLC62SC6Y/C2969755) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2211080730_STMicroelectronics-USBLC6-2SC6Y_C2969755.pdf) |
| 1 | KH-TYPE-C-16P | J4 | KH-TYPE-C-16P | SMD | USB Type-C Connector | [Link](https://jlcpcb.com/partdetail/Shenzhen_KinghelmElec-KH_TYPE_C16P/C709357) | [Datasheet](https://www.lcsc.com/datasheet/C709357.pdf) |
| 1 | 5034802400 | J1 | 5034802400 | SMD, P=0.5mm | Molex Display Connector | [Link](https://jlcpcb.com/partdetail/MOLEX-5034802400/C122434) | [Datasheet](https://www.molex.com/content/dam/molex/molex-dot-com/products/automated/en-us/salesdrawingpdf/503/503480/5034802400_sd.pdf) |
| 1 | MAX17048G+T10 | U3 | MAX17048G+T10 | DFN-8-EP(2x2) | Fuel Gauge | [Link](https://jlcpcb.com/partdetail/2777647-MAX17048GT10/C2682616) | [Datasheet](https://www.lcsc.com/datasheet/C2682616.pdf) |
| 3 | EVPAKE31A | SW_UP, SW_ENT, SW_DN | EVPAKE31A | SMD,3.9x2.9mm | Push Button | [Link](https://jlcpcb.com/partdetail/PANASONIC-EVPAKE31A/C569760) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2301111010_PANASONIC-EVPAKE31A_C569760.pdf) |
| 1 | DRV2605YZFR | IC2 | DRV2605YZFR | DSBGA-9 | Haptic Driver | [Link](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C81079) | [Datasheet](https://www.ti.com/cn/lit/ds/symlink/drv2605.pdf?ts=1775581743191&ref_url=https%253A%252F%252Fjlcpcb.com%252F) |
| 1 | DMG2305UX-7 | Q1 | DMG2305UX-7 | SOT-23 | EPD Power Gate | [Link](https://jlcpcb.com/partdetail/DiodesIncorporated-DMG2305UX7/C150470) | [Datasheet](https://www.lcsc.com/datasheet/C150470.pdf) |
| 1 | SI1308EDL-T1-GE3 | Q3 | SI1308EDL-T1-GE3 | SOT-323 | EPD Boost Converter Gate | [Link](https://jlcpcb.com/partdetail/VishayIntertech-SI1308EDL_T1GE3/C469327) | [Datasheet](https://www.lcsc.com/datasheet/C469327.pdf) |
| 3 | MBR0530 | D2, D4, D5 | MBR0530 | SOD-123 | EPD Boost Converter Schottky Diode | [Link](https://jlcpcb.com/partdetail/Yangzhou_Yangjie_ElecTech-MBR0530/C699108) | [Datasheet](https://www.lcsc.com/datasheet/C699108.pdf) |
| 1 | BMA421 | IC3 | BMA421 | LGA-12(2x2) | Accelerometer (IMU) | [Link](https://jlcpcb.com/partdetail/BoschSensortec-BMA421/C5242966) | [Datasheet](https://files.pine64.org/doc/datasheet/pinetime/BST-BMA421-FL000.pdf) |
| 1 | RT6160AWSC | IC9 | RT6160AWSC | WLCSP-15B(2.3x1.4) | DC/DC Converter | [Link](https://jlcpcb.com/partdetail/RichtekTech-RT6160AWSC/C7065276) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2312271436_Richtek-Tech-RT6160AWSC_C7065276.pdf) |
| 1 | BQ25180YBGR | IC1 | BQ25180YBGR | DSBGA-8(1.1x1.6) | LiPo Charger | [Link](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C3682423) | [Datasheet](https://www.ti.com/cn/lit/ds/symlink/bq25180.pdf?ts=1775521983580) |
| 1 | 2450AT18B100E | ANT1 | 2450AT18B100E | 1206 | RF Antenna | [Link](https://jlcpcb.com/partdetail/JohansonDielectrics-2450AT18B100E/C2917717) | [Datasheet](https://www.lcsc.com/datasheet/C2917717.pdf) |
| 1 | NRF52840-QIAA-R | U1 | NRF52840-QIAA-R | AQFN-73-EP(7x7) | Microcontroller (MCU) | [Link](https://jlcpcb.com/partdetail/NordicSemicon-NRF52840_QIAAR/C190794) | [Datasheet](https://www.lcsc.com/datasheet/C190794.pdf) |
| 1 | X1A0000910005 | X2 | X1A0000910005 | SMD3215-2P | Low Frequency Crystal | [Link](https://jlcpcb.com/partdetail/SeikoEpson-X1A0000910005/C255869) | [Datasheet](https://wmsc.lcsc.com/wmsc/upload/file/pdf/v2/lcsc/2008251105_Seiko-Epson-X1A0000910005_C255869.pdf) |
| 1 | Q22FA12800025 | X1 | Q22FA12800025 | SMD2016-4P | High Frequency Crystal | [Link](https://jlcpcb.com/partdetail/SeikoEpson-Q22FA12800025/C187794) | [Datasheet](https://www.lcsc.com/datasheet/C187794.pdf) |
| 1 | AKYGA LP502030 | - | AKYGA LP502030 | - | 3.7V/250mAh LiPo Battery | [Link](https://www.tme.eu/dk/en/details/aky-lp502030/rechargeable-batteries/akyga-battery/aky0106/) | [Datasheet](https://www.tme.eu/Document/b9e12bf26ad0ba929a22ab5d58f022cd/AKY0106.pdf) |
| 1 | WSH-12561 | - | WSH-12561 | - | 1.54" E-Paper Display, 200x200px | [Link](https://www.tme.eu/ie/en/details/wsh-12561/e-paper/waveshare/12561/) | [Datasheet](https://www.tme.eu/Document/0ca57a8ffbcd57b5bca53252eb9d6ec3/WSH-12561.pdf) |
| 1 | FIT0774 | - | FIT0774 | - | Shaker | [Link](https://www.tme.eu/ro/details/df-fit0774/motoare-dc/dfrobot/fit0774/) | [Datasheet](https://www.mouser.com/pdfDocs/ProductOverview_DFRobot-FIT0774.pdf) |
| 1 | TC2030-IDC | J2 | TC2030-IDC | PCB Footprint | Debug Connector | - | - |

## License
This project is licensed under the **MIT License**. See the `LICENSE` file for details.