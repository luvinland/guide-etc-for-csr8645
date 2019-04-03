# Get Started
CSR8645 이용한 VP 적용 방법 가이드.

### Step 1. VP wav 파일 저장 폴더
* `d:\00_LG_Tone_VP\` 등 폴더이름은 간략히.

### Step 2. VP wav 파일 이름
* `9041-K.wav` 등 파일이름도 간략히.

### Step 3. 파일명 관리
* 파일명과 VP 매칭되도록 관리.

   ![01](https://user-images.githubusercontent.com/26864945/55456826-08068580-5623-11e9-920b-fb1611747700.PNG)

### Step 4. `Bootmode1.psr` 파일
* Config tool 에서 최신 Bootmode1.psr 파일 Open from File.

### Step 5. Config Tool 적용
* VP 탭에서 VP 수정, 추가, 또는 삭제.

   ![02](https://user-images.githubusercontent.com/26864945/55457002-911dbc80-5623-11e9-8c3b-6a05b21311ee.PNG)
   
### Step 6. Download
* SPI 연결 후, Download Prompts to Device → Yes → Waiting → End.

### Step 7. Binary `dump.xuv` 확보.
```c
"C:\CSR\BlueSuite 2.6.2\nvscmd.exe" -usb 0 dump "dump.xuv"
```
