# LTPI Study

## 🎯 Objective
- Record and organize studies related to Low-Pin Interface (LTPI) for BMC development.
- Compare differences between AST2700 (SCM) and AST1700 (HPM).

## 📖 Key Learnings
- **Pin Definition**:
    - [AST1700 Pin Definition & Statistics](LTPI_AST1700_PIN_DEFINITION.md)
- **LTPI Controller**: Detailed register and base address comparison between AST2700 and AST1700.
- **Data Channels**: Comparing protocol mappings for:
    - [ADC](LTPI_DATA_CH_ADC.md)
    - [GPIO (Parallel vs. Non-Latency)](LTPI_DATA_CH_GPIO.md)
    - [I2C](LTPI_DATA_CH_I2C.md)
    - [I3C](LTPI_DATA_CH_I3C.md)
    - [JTAG](LTPI_DATA_CH_JTAG.md)
    - [LTPI CTRL](LTPI_DATA_CH_LTPI_CTRL.md)
    - [PWM/TACHO](LTPI_DATA_CH_PWM_TACHO.md)
    - [SGPIO](LTPI_DATA_CH_SGPIO.md)
    - [WDT](LTPI_DATA_CH_WDT.md)

## 🛠️ Implementation Details
- Comparison based on AST2700 and AST1700 technical documentation.
- Using DTS compatible strings for characterization.

## 🔗 Reference Links
- [TBD]

## 📝 Status & Next Steps
- [ ] Task: Deep dive into LTPI Link Training sequence.
- [x] Done: AST1700 Pin statistics and detailed channel mapping.
