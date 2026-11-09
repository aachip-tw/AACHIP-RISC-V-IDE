# 驗證紀錄

日期：2026-11-06。使用本工作目錄目前的Mode3／Timer1 UART SOF、50MHz CPU、外部512KB程式SRAM＋16KB BRAM。

**36組純C#建置（12專案×O0/O2/O3）及12個O0實板流程，共122項檢查通過。** O2/O3本次僅編譯、輸出及來源行號檢查，沒有宣稱其實板執行全部通過。

| 範例 | 編譯 | 實板內容與範圍 |
|---|---|---|
| [01_LED_Blink](01_LED_Blink/README.md) | O0／O2／O3通過 | O0實板：P0.0切換，GPIO實體回讀符合輸出；未替外接LED接線量測亮度。 |
| [02_Running_Light](02_Running_Light/README.md) | O0／O2／O3通過 | O0實板：one-hot跑馬燈與實體腳位回讀通過。 |
| [03_P0_Counter_UART](03_P0_Counter_UART/README.md) | O0／O2／O3通過 | O0實板：01、02、FF、00輸出與UART迴路一致，來源斷點可停。 |
| [04_Key_Debounce](04_Key_Debounce/README.md) | O0／O2／O3通過 | O0實板：以P3.2輸出回讀模擬穩定按下／放開；按住不重複、再按加1。未接實體機械按鍵驗證彈跳波形。 |
| [05_Timer0_Poll](05_Timer0_Poll/README.md) | O0／O2／O3通過 | O0實板：mode1重載、tick累積通過；未使用儀器量測絕對週期。 |
| [06_Dual_Timer_IRQ](06_Dual_Timer_IRQ/README.md) | O0／O2／O3通過 | O0實板：雙ISR返回主程式、1ms與2ms計數先後通過；未量測精確抖動。 |
| [07_External_Counter](07_External_Counter/README.md) | O0／O2／O3通過 | O0實板：P3.4輸出回讀產生三次下降緣，C程式count與P0皆為3。未用外部脈波源量測最高頻率。 |
| [08_INT_Priority](08_INT_Priority/README.md) | O0／O2／O3通過 | O0實板：P3.2/P3.3輸出回讀產生同步pending，INT1先仲裁，兩個ISR皆清旗標並返回。未接實體按鍵。 |
| [09_UART_Echo](09_UART_Echo/README.md) | O0／O2／O3通過 | O0實板：以P3.6/P3.7迴路注入5A，C程式接收、P0輸出及回送通過。未使用外部PC終端作連續資料測試。 |
| [10_Timer1_UART_Baud](10_Timer1_UART_Baud/README.md) | O0／O2／O3通過 | O0實板：Timer1 mode2/15625設定及ASCII訊息含LF迴路通過；未用外部儀器獨立量測鮑率精度。 |
| [11_Mode3_Split](11_Mode3_Split/README.md) | O0／O2／O3通過 | O0實板：TL0與TH0各自溢位、GPIO實體回讀通過。 |
| [12_UART_Loopback_Test](12_UART_Loopback_Test/README.md) | O0／O2／O3通過 | O0實板：done=1、pass_count=256、error_count=0，00..FF所有byte通過實體跳線。 |

## 實板測試方法

UART使用已接好的P3.6→P3.7實體跳線。按鍵／Counter／INT測試由FPGA GPIO輸出驅動同一腳位，再經實際pad輸入回讀，無額外外部訊號源；教材正常使用時仍將它們設為輸入。這驗證了C程式、MMIO及FPGA周邊整合，不能取代實體按鍵接點雜訊、外部最高輸入頻率或電氣規格量測。

測試結束：CPU暫停，Timer與周邊中斷停用，UART恢復115200，暫時使用的P3.2..P3.5驅動還原。最後下載的是12_UART_Loopback_Test，自測已完成。

詳細原始紀錄在原開發工作目錄：test-artifacts/learning8051-tests.txt、learning8051-build.txt。測試入口：Aachip.RiscV.Tests --learning8051（編譯）或--hardware-learning8051（編譯＋需已接UART迴路的實板測試）。

## 繁體中文註解補充

已為12個main.c及common/aachip8051.h共13個檔案補上教學註解。修改前後C token完全一致；重新進行12×O0/O2/O3共36組建置及來源資訊檢查，79項通過。12個O0 firmware.bin的SHA256逐一與註解前相同，確認註解未改變執行指令。本次未重複實板測試，沿用前述相同指令內容的實板結果。

註解新增後來源行號已改變，請F7重新編譯並重新下載／載入ELF，再重新選取斷點行。使用SLN重新建置時，IDE輸出目錄中的教學檔案也會同步更新。
