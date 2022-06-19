# Merlin Firmware update

## Arduino IDE

### Install Arduino IDE
RaspberryPI:
```bash
sudo apt install arduino
```
Windows: Install from Microsoft Store.

### Configure
- Select Board from Tools > Board: "Arduino/Genuino Mega or Mega 2560"
- Select Processor from Tools > Processor: "ATmega2560 (Mega 2560)"
- Select Port from Tools > Port: 
- Select Programmer from Tools > Programmer: "Arduino as ISP"

### Upload to printer
- Make sure there ar no active connections to the board (i.e. OctoPrint)
- Click the Verify button at the top of the window
- Click Upload to flash firmware to board
