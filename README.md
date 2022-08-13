# Python透過Serial取得量測儀器RealTime Data



```python
# Windows 10 64-bit
# Python 3.9.7 
# HW: USB UART Dongle CP2102
# CTRL Device: Keithley multi 2000
# Connect Wire: RS232 to USB (CH340)
# Protocol: SCPI (Standard Commands for Programmable Instruments)

import serial
import threading

import tkinter as tk
from tkinter import filedialog


# ==================== UART Parameter Config Region =================
Kethley_uart = serial.Serial()
Kethley_uart.port = "COM6"
Kethley_uart.baudrate = 19200
Kethley_uart.bytesize = serial.EIGHTBITS  # number of bits per bytes
Kethley_uart.parity = serial.PARITY_NONE  # set parity check
Kethley_uart.stopbits = serial.STOPBITS_ONE  # number of stop bits
Kethley_uart.timeout = 0.5  # non-block read 0.5s
Kethley_uart.writeTimeout = 0.5  # timeout for write 0.5s
Kethley_uart.xonxoff = False  # disable software flow control
Kethley_uart.rtscts = False  # disable hardware (RTS/CTS) flow control
Kethley_uart.dsrdtr = False  # disable hardware (DSR/DTR) flow control


#========== Global variable Define ============
Kethley_uart_rx_buffer = b''


def clr_Kethley_uart_rx_buffer():
    global Kethley_uart_rx_buffer
    Kethley_uart_rx_buffer = b''


def Window_on_Close():
    global win
    print("Windows close")
    win.destroy()

def TxButton_click():
    w_buffer = b'MEASure:VOLTage:DC?\n'    # MEASure:VOLTage:DC?
    Kethley_uart.write(w_buffer)
    pass

def Master():    
    print("<Master ID>:", threading.get_ident())   


    global win
    win = tk.Tk()
    win.title("電壓紀錄器")
    #==========  UI code here ====================
    TxButton = tk.Button(text="傳送",command=TxButton_click)
    TxButton.pack()
    #=========================================
    win.protocol("WM_DELETE_WINDOW", Window_on_Close)
    win.mainloop()



# ================ UART RX ===============================
def Kethley_UartRxBack():  

    global Kethley_uart_rx_buffer     
    print("<Kethley_UartRxBack ID>:", threading.get_ident())

    while 1:
        
        try:
            read_byte = Kethley_uart.read(1)
        except:
            read_byte = b''
        
        if(read_byte != b'\r'):
            Kethley_uart_rx_buffer += read_byte
        elif ((read_byte == b'\r')):
            # 偵測到 '\n' 換行符號 用utf8解析，並print出來
            string_buffer = str( Kethley_uart_rx_buffer , encoding = "utf-8" )
            #print(string_buffer)
            value = float(string_buffer)
            print(value)


            # 資料包解析完成 清空 Kethley_uart_rx_buffer 才能接收下一包資料
            clr_Kethley_uart_rx_buffer()

# ================ UART RX ====================================

if __name__ == "__main__":
    
    print("<Main ID>:", threading.get_ident())

    try:
        Kethley_uart.open()
        print("open Kethley Serial OK ")

        # ===== UART TX ==========
        #w_buffer = b'MEASure:VOLTage:DC?\n'    # MEASure:VOLTage:DC?
        #uart.write(w_buffer)
        # ===== UART TX ==========

    except Exception as ex:
        print("open Kethley error " + str(ex))
        exit()

   

    task_1 = threading.Thread(target=Master)
    task_2 = threading.Thread(target=Kethley_UartRxBack)

    # 如果有寫UI才需要把 task_2.setDaemon 設 True (功能:視窗關閉時，同時結束task_2)
    task_2.setDaemon(True)

    task_1.start()
    task_2.start()
```

