# Get Started
CSR8645 이용한 바이너리 최적화 방법 가이드. Release 버전의 불필요한 조각모음 동작을 줄이고자 함.

### Step 1. Erase EEPROM
```
"C:\CSR\BlueSuite 2.6.2\nvscmd.exe" -usb 0 erase
```

### Step 2. CSR8645 `default.psr` 및 `default.xuv` 확보
```
"C:\CSR\BlueSuite 2.6.2\nvscmd.exe" -usb 0 dump "default.xuv"
```

### Step 3. `user.psr` 작성
* default.psr 과 적용하고자 하는 ???.psr 비교하여 필요부분만 별도의 user.psr 로 작성.
   >중복 write 되어 불필요한 메모리 사용을 방지하기 위해.

### Step 4. VP Data 추출
* VP 가 올바르게 적용된 dump.xuv 확보하여 @002000 이후 데이터만 추출.
* VP 데이터만 사용하고자 함으로 `dump_vp.xuv` 로 저장. (~ @001fff 데이터는 불필요)

### Step 5. Default + VP Data
* `default.xuv` 파일을 편집기에서 (sourceinsight 등) open 하여, @002000 이후 데이터는 `dump_vp.xuv` 내용과 merge 한 후 `CSR8645_default_vp.xuv` 이름으로 저장.

### Step 6. CSR8645_default_vp.xuv 다운로드.
* merge 한 CSR8645_default_vp.xuv 를 보드에 다운로드.

   ```
   "C:\CSR\BlueSuite 2.6.2\nvscmd.exe" -usb 0 burn "CSR8645_default_vp.xuv"
   ```
   
### Step 7. `user.psr` merge
* 보드에 SPI 연결, PSTool 사용하여 user.psr merge. (Bootmode 고려)

### Step 8. Release Binary 확보.
* dump 하여 신규버전 binary 획득.

   ```
   "C:\CSR\BlueSuite 2.6.2\nvscmd.exe" -usb 0 dump "release.xuv"
   ```
