# 8051 使用者的 RISC-V C 入門範例

預設最佳化為 **-O2**。需要保留較完整的參數／區域變數與單步流程時，可在 IDE「編譯 → C99 / 函式庫 / 體積最佳化…」手動選擇 **-O0** 並儲存；之後會保留該選擇。

適用目前 MAGIC1 FPGA：外部512KB程式SRAM＋16KB資料BRAM，含GPIO／Timer8051／Mode3／Timer1 UART鮑率的SOF。所有範例是純C# Aachip C compiler/assembler/linker的RV32IM/ILP32專案，不需GCC/GDB；不是8051二進位模擬。

## 開始使用

1. 保留整個Learn8051_RiscV目錄，個別專案會引用共用的common/aachip8051.h。
2. IDE「檔案 → 8051 常用範例（RISC-V C）」選擇專案，或直接開啟所選.rvproj，按F7編譯。
3. 選FT2232H Port B，連線後下載ELF。Port A供SOF燒錄；學習範例不需重燒SOF。
4. 在main.c實際敘述行設斷點，使用C SOURCE DEBUG的RUN、STEP IN／OVER／OUT及Watch。
5. 下列程式多為無窮迴圈；RUN後持續執行是正常行為，需斷點或STOP才停止。除錯暫停預設凍結Timer。

| 教學 | 專案 |
|---|---|
| [01 LED 閃爍／P0 位元操作](01_LED_Blink/README.md) | `01_LED_Blink/01_LED_Blink.rvproj` |
| [02 八位跑馬燈／循環位移](02_Running_Light/README.md) | `02_Running_Light/02_Running_Light.rvproj` |
| [03 P0 加1後傳送 UART](03_P0_Counter_UART/README.md) | `03_P0_Counter_UART/03_P0_Counter_UART.rvproj` |
| [04 按鍵輸入／20ms 去彈跳](04_Key_Debounce/README.md) | `04_Key_Debounce/04_Key_Debounce.rvproj` |
| [05 Timer0 mode1 輪詢計時](05_Timer0_Poll/README.md) | `05_Timer0_Poll/05_Timer0_Poll.rvproj` |
| [06 Timer0／1 自動重載中斷](06_Dual_Timer_IRQ/README.md) | `06_Dual_Timer_IRQ/06_Dual_Timer_IRQ.rvproj` |
| [07 T0 外部下降緣計數](07_External_Counter/README.md) | `07_External_Counter/07_External_Counter.rvproj` |
| [08 INT0／INT1 與兩級優先序](08_INT_Priority/README.md) | `08_INT_Priority/08_INT_Priority.rvproj` |
| [09 UART 接收與回送](09_UART_Echo/README.md) | `09_UART_Echo/09_UART_Echo.rvproj` |
| [10 Timer1 產生 UART 鮑率](10_Timer1_UART_Baud/README.md) | `10_Timer1_UART_Baud/10_Timer1_UART_Baud.rvproj` |
| [11 Mode3 分拆兩個8位 Timer](11_Mode3_Split/README.md) | `11_Mode3_Split/11_Mode3_Split.rvproj` |
| [12 UART 256-byte 實體迴路自測](12_UART_Loopback_Test/README.md) | `12_UART_Loopback_Test/12_UART_Loopback_Test.rvproj` |

## 8051 對照

| 8051習慣 | 本FPGA的RISC-V C方式 |
|---|---|
| P0 | P0／P0_OUT 的 bit0..7，P0_DIR 設定輸入／輸出 |
| P3 | P3／P3_OUT 的 bit0..7；P0～P3 各自獨立 |
| sbit P0_0 | 使用遮罩／位元運算，沒有sbit關鍵字 |
| P0輸入／準雙向埠 | 本GPIO是明確方向，對應 Pn_DIR 位元=0才是輸入，並非寫1就輸入 |
| TXD/RXD通常在P3.1/P3.0 | 此板自訂P3.6=TX、P3.7=RX；GPIO_ALT bit0=1啟用 |
| INT0/INT1/T0/T1 | P3.2/P3.3/P3.4/P3.5 |
| TCON/TMOD/TH/TL | 本表頭的32-bit MMIO，位元功能對應，位址不是8051 SFR |
| EA/ET0等與interrupt n | 同時設定周邊IE及RISC-V mie.MEIE/mstatus.MIE；C ISR使用interrupt("machine") |
| TF0/TF1由ISR入口自動清除 | 此設計必須明確寫IRQ_ACK；1代表清除對應來源 |
| ISR向量與RETI | startup固定0x80000020跳至aachip_machine_trap，編譯器輸出mret |
| 8051 ROM/RAM獨立空間 | 程式0x80000000、資料0x40000000，為同一RISC-V位址空間的不同區域 |

## 共用暫存器及限制

完整位址與函式在common/aachip8051.h。UART DATA=0xF0000000、STATUS=+4；P0 OUT/IN/DIR=0xF0000140/144/148，P1～P3 每組加0x10；Timer/INT=0xF0000200；UART_CLOCK=0xF0000230。

board_init會停用中斷、停止Timer、恢復UART固定115200、啟用P0輸出，並明確讀取一次殘留RX來清旗標。它保留P1/P2及其餘GPIO的設定；各輸入範例另將所需P3腳位設為輸入。

所有MMIO以volatile存取。ISR與主程式共享的計數器也用volatile；但volatile不等於複合運算的原子性。此教學中ISR負責增加計數，主程式只讀取它們。

Timer時脈為50MHz / CLOCK.divider（0視為1）。Mode2週期=(256-TH)/Timer頻率；mode1需軟體重載。延遲、輪詢與ISR執行都有開銷，不能把示範時間視為量測儀器精度。IRQ旗標只有1bit，長時間阻塞可能合併多個事件。

IP僅提供兩級來源仲裁，無完整8051中斷巢狀。UART為8-N-1、單一接收暫存器，沒有FIFO／UART中斷；09範例採單byte送出後等待回送。

接外部模組時使用符合板卡的邏輯電位並共地；不可把RS-232正負電壓直接接FPGA GPIO。LED需限流，按鍵輸入需上拉，按鍵彈跳與乾淨脈波不同。

## 常見問題

遇到RUN不停、UART顯示亂碼、Timer暫停等問題，請看 [FAQ.md](FAQ.md)。

## 驗證與分享

[VALIDATION.md](VALIDATION.md)逐一列出編譯／實板驗證結果，未接實體按鍵或外部脈波的情況會明確標示。所有範例有繁體中文註解及獨立README。分享時請提供整個Learn8051_RiscV目錄，不要只複製單一main.c。

### 原始碼註解

每個main.c開頭說明教學目標、接線、流程及Watch；重要宣告與操作補上繁體中文註解。共用標頭也逐項說明位址、旗標、ACK與IRQ編號，以及阻塞／非阻塞收發函式。閱讀時可先從main開始，再跳到common/aachip8051.h查看函式實作。

## 完整程式中文說明

12個範例的流程、設定值及預期結果見 [程式中文導讀](程式中文導讀.md)。
