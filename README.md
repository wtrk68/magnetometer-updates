```mermaid
graph LR
    %% POWER SUPPLY ----------------------------------------------------
    subgraph Power ["Power (Li-Ion / Li-Po 3.7V)"]
        Vbat[+ Vbat 3.7V] 
        GNDbat[(GND)]
    end

    %% MOSFET HIGH-SIDE (P-MOS) ----------------------------------------
    subgraph Pmos ["P-MOSFET (Si2301) - High-Side Switch"]
        GateP[Gate] 
        SourceP[Source] 
        DrainP[Drain] 
        GateP -->|R1 100 kΩ Pull-up| SourceP
    end

    %% LDO REGULATOR ---------------------------------------------------
    subgraph LDO_Block ["3.3V LDO - MCP1700-33"]
        Vin[Vin] 
        Vout[Vout 3.3V]
        GNDldo[(GND)]
    end

    %% LATCH DRIVER (N-MOS) ---------------------------------------------
    subgraph Nmos ["N-MOSFET (BSS138) - Latch Driver"]
        GateN[Gate] 
        DrainN[Drain] 
        SourceN[Source] 
    end

    %% Schmitt-Trigger (Power-OFF detector) -----------------------------
    subgraph Schmitt ["Schmitt-Trigger 74LVC1G17"]
        VinTrig[Vin] 
        VoutTrig[Vout] 
    end

    %% ESP32 MCU ---------------------------------------------------------
    subgraph MCU ["ESP32-DevKit"]
        ESP32_EN[(EN-LATCH GPIO5)] 
        ESP32_PWR_OFF[(POWER_OFF GPIO7 INPUT)] 
        ESP32_GREEN[(GREEN LED GPIO5)] 
        ESP32_BLUE[(BLUE LED GPIO6)] 
        ESP32_I2C_SDA[(I2C SDA GPIO21)]
        ESP32_I2C_SCL[(I2C SCL GPIO22)]
    end

    %% ADS1115 ADC ---------------------------------------------------------
    subgraph ADC ["ADS1115 ADC"]
        ADS_VCC[(VCC 3.3V)] 
        ADS_GND[(GND)] 
        ADS_SDA[(SDA)] 
        ADS_SCL[(SCL)] 
        ADS_A0[(A0)] 
        ADS_A1[(A1)] 
        ADS_A2[(A2)] 
        ADS_A3[(A3)] 
    end

    %% SENSORS -------------------------------------------------------------
    subgraph Sensors ["Gradiometer / Hall Sensors"]
        S1_OUT[(Sensor-1 OUT)] 
        S1_GND[(Sensor-1 GND)] 
        S2_OUT[(Sensor-2 OUT)] 
        S2_GND[(Sensor-2 GND)] 
    end

    %% BUTTON --------------------------------------------------------------
    ButtonNC[NC-Button / Power-On]

    %% CRITICAL ELECTRICAL CONNECTIONS -------------------------------------
    Vbat --> SourceP
    DrainP -->|Switched 3.7V| Vin
    Vout -->|VCC 3.3V Rail| ADS_VCC
    Vout -->|VCC 3.3V Rail| MCU
    
    GNDbat --> GNDldo
    GNDldo --> SourceN
    GNDldo --> ADS_GND
    GNDldo --> S1_GND
    GNDldo --> S2_GND

    ButtonNC -->|Press: Pulls Gate to GND| GateP
    ESP32_EN -->|High: Keeps Power On| GateN
    DrainN -->|Pulls Gate to GND| GateP

    ButtonNC -->|RC Filter: R2 10k + C1 100µF| VinTrig
    VoutTrig --> ESP32_PWR_OFF

    ESP32_EN --> ESP32_GREEN
    Vout --> ESP32_BLUE

    S1_OUT --> ADS_A0
    S2_OUT --> ADS_A2
    GNDldo --> ADS_A1
    GNDldo --> ADS_A3

    ADS_SDA <--> ESP32_I2C_SDA
    ADS_SCL <--> ESP32_I2C_SCL

    %% STYLES --------------------------------------------------------------
    style Power fill:#f9f9f9,stroke:#333,stroke-width:1px
    style LDO_Block fill:#e0f7fa,stroke:#006064,stroke-width:2px
    style Pmos fill:#fff3e0,stroke:#e65100,stroke-width:2px
    style Nmos fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    style Schmitt fill:#e1f5fe,stroke:#0277bd,stroke-width:2px
    style MCU fill:#fffde7,stroke:#f57f17,stroke-width:2px
    style ADC fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    style Sensors fill:#fce4ec,stroke:#ad1457,stroke-width:2px
    style ButtonNC fill:#ffebee,stroke:#c62828,stroke-width:2px
