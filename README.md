# AACHIP-RISC-V-IDE
A Windows RISC-V IDE with a pure AACHIP C  C99 compiler, assembler and debugger, direct FTDI JTAG FPGA debugging, offline simulation, 8051-style peripherals and configurable register maps.
AACHIP RISC-V IDE
AACHIP RISC-V IDE 是面向 Windows 與 FPGA 開發的 RISC-V 整合開發環境，整合程式編輯、C 編譯、組譯、ELF 下載、原始碼除錯、記憶體檢視與周邊暫存器操作。

本專案的一項核心特色，是以 C99 實作編譯器、組譯器與原生除錯後端，並將 FPGA 硬體除錯與離線軟體模擬整合在同一套操作介面中。
主要特色
純 C 編譯與除錯工具鏈
內建 C99 編譯器、RISC-V 組譯與 ELF／DWARF 處理功能。內建編譯流程不需呼叫 GCC、GNU assembler、linker 或 objcopy；原生除錯後端也不需啟動 GDB。

另外提供外部 GCC 工具鏈設定，方便依專案需求選擇。
C 語言相容性以實際支援範圍為準，不宣稱完整支援所有 C99 功能。

同一介面支援 EMULATION 與 SIMULATION
- EMULATION（綠色）：透過 FTDI MPSSE／JTAG 與 FPGA 上的 RISC-V 除錯模組通訊。
- SIMULATION（紅色）：使用內建 RV32IM 功能模擬器，不需要連接 FPGA。
- 未連線時可直接使用 RUN、STEP 或 RESET 進入軟體模擬。
- 中斷硬體連線後，可透過 RESET 啟動 Simulation。
模擬器適合驗證程式流程、暫存器與記憶體操作；硬體時序、UART 時序及外部中斷訊號仍需在實體 FPGA 上驗證。
直接透過 FTDI 進行 FPGA 除錯
原生硬體除錯路徑直接透過 FTDI D2XX／MPSSE 存取 RISC-V Debug Module，不必另外啟動 OpenOCD 或 GDB。

支援 ELF 下載、執行、暫停、重設、單步與硬體斷點。硬體連線流程包含自動下載 ELF，減少忘記下載程式而執行舊內容的情況。
C 原始碼與組合語言同步除錯
提供 C Source Debug 與組合語言檢視，可依除錯資訊定位目前執行位置，並操作：
- 原始碼與指令單步。
- Step Over、Step Out。
- 來源行及指令斷點。
- CPU 暫存器、CSR 狀態、Watch 與呼叫堆疊。
- 專案實際使用的 C／H 檔案檢視。
目前對應 FPGA 設計提供最多 3 個硬體斷點。
RISC-V 搭配 8051 風格周邊
搭配的 FPGA 系統提供 8051 風格的 GPIO、Timer 與中斷控制暫存器，讓熟悉 8051 周邊操作方式的使用者，可以逐步轉向 RISC-V。

包含 P0～P3、GPIO 輸入／輸出與方向控制、Timer0／Timer1、中斷控制及 UART 相關暫存器。

這是 RISC-V CPU 搭配 8051 風格周邊，不是執行 8051 指令的相容核心。
透過 reg.ini 自訂 IO 清單
右側 IO 暫存器表由外部 reg.ini 定義，可設定：
- 暫存器名稱、位址與中文說明。
- RO、RW、WO、W1C 存取類型。
- 寫入遮罩。
- 是否自動讀取。
FPGA 增加暫存器後，可修改設定檔並在 CPU 暫停時重新載入，不必為每個新暫存器重新修改 IDE。

設定載入順序：
專案資料夾的 reg.ini
        ↓
C:\AACHIP\RiscV\reg.ini
        ↓
內建 8051 IO 設定
目前使用對齊的 32-bit MMIO 暫存器。設定檔僅描述介面，對應功能仍須由 FPGA RTL 實作。
避免自動讀取造成周邊副作用
有些暫存器在讀取時會改變硬體狀態。例如 UART DATA 讀取可能清除 RX valid，因此可設定為手動讀取，避免畫面更新意外消耗接收資料。
分離式程式與資料記憶體檢視
支援對應 FPGA 的程式／資料分離配置，提供 INROM、INRAM HEX 檢視、編輯及寫入回讀檢查，方便觀察程式與資料變化。
中文學習範例
提供 8051 風格周邊的 RISC-V C 範例與中文說明，包括 LED、流水燈、按鍵、計時器、中斷與 UART 等主題。
Release Native AOT 單檔交付
Windows x64 發佈版本使用 Native AOT，無需另外安裝 .NET Runtime。

預設安裝至：C:\AACHIP\RiscV\
使用需求
- Windows x64。
- 離線 Simulation 不需要 FPGA 或 FTDI 驅動, 不需要授權。
- 硬體除錯需相容的 FPGA 設計、需使用本公司專用的FTDI、接線與有效授權。
- FPGA 連線需另外安裝 FTDI 驅動；`FTD2XX.dll` 不等於 Windows 驅動。

目前硬體支援以搭配的 AACHIP／MAGIC1 FPGA 系統為主，其他 RISC-V 平台需確認記憶體配置、Debug Module 與周邊協定。
第三方元件與授權
本產品包含第三方元件，各元件依其原始授權條款使用。

VexRiscv 採 MIT License；若發布內容包含 VexRiscv 或其衍生成果，請一併提供適用的著作權聲明與完整授權文字，詳見 THIRD_PARTY_NOTICES.txt。

AACHIP 自行開發內容的使用範圍，依本倉庫正式公布的授權條款為準。
