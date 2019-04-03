# Get Started
CSR867x 이용한 OTA VP 적용 방법 가이드.

## Step 1. VP 데이터 파일 (\*.idx, \*.prm) 생성
1. Language 개수 → Event mapping → Generate

   ![01](https://user-images.githubusercontent.com/26864945/55311980-5854da80-549f-11e9-9773-55d2b6e4e1a4.PNG)

1. `app\sink\image` 폴더에 생성된 `\header`, `\prompts`, `\refname` 폴더 이동 (잘라내기)
   1. 임의 지정 폴더로 이동 (잘라내기) (예. `app\sink\audioprompts`)
   
   1. Language 단위로 폴더 분리   
      예.) 4개 language 의 경우, `\01`, `\02`, `\03`, `\04` 폴더 생성  
      → 각각의 `\header`, `\prompts` 폴더 생성 및 데이터 이동. (`\01` : 0~15, `\02` : 16 ~ 31, …)
      
         ![02](https://user-images.githubusercontent.com/26864945/55312009-67d42380-549f-11e9-8325-9265c007c2ad.PNG)

1. VP 데이터 폴더를 xuv 파일로 변경. (packing) : 아래 명령어 이용하여 4개 xuv 생성
   ```c
   \tools\bin\packfile.exe 01 ptn01.xuv
   \tools\bin\packfile.exe 02 ptn02.xuv
   \tools\bin\packfile.exe 03 ptn03.xuv
   \tools\bin\packfile.exe 04 ptn04.xuv
   ```

## Step 2. 기본 파티션 생성.
1. 파티션 설정 파일 작성. `vp.ptn`
   ```c
   0, 64K, RO, ptn01.xuv  # 첫 번째 언어 (위 1-C-i. 에서 생성한 xuv 파일)
   1, 64K, RO, ptn02.xuv
   2, 64K, RO, ptn03.xuv
   3, 64K, RO, ptn04.xuv
   4, 64K, RO, (none)     # 첫 번째 언어의 OTA 를 위한 공간
   5, 64K, RO, (none)
   6, 64K, RO, (none)
   7, 64K, RO, (none)
   ```

1. 외부 메모리 초기화
   ```c
   nvscmd.exe –usb 0 erase
   ```

1. 외부 메모리 파티션 write. `vp.ptn` 참조
   ```c
   nvscmd.exe –usb 0 burn vp.ptn all
   ```

1. `Sink_upgrade.c` (`logicalPartitions[]` 배열 수정)
   ```c
   static const UPGRADE_UPGRADABLE_PARTITION_T logicalPartitions[] = {
      UPGRADE_PARTITION_DOUBLE(0x1000,0x1004,MOUNTED),  // 첫 번째 언어 공간
      UPGRADE_PARTITION_DOUBLE(0x1001,0x1005,MOUNTED),
      UPGRADE_PARTITION_DOUBLE(0x1002,0x1006,MOUNTED),
      UPGRADE_PARTITION_DOUBLE(0x1003,0x1007,MOUNTED)
   };
   ```

## Step 3. XIDE sink project 설정 변경.
1. Define symbols : `ENABLE_SQIFVP`

1. OTA App. GAIA Control 버전에 따라 GAIA SPP 활성화 필요함.

   1. GAIA Control v3 의 경우 XIDE GAIA SPP feature : `Enabled`
   
   1. `Gaia_private.h` (define 추가, line 15) → VM lib. clean → build.
      ```c
      #define GAIA_TRANSPORT_NO_RFCOMM 1
      #define GAIA_TRANSPORT_SPP 1
      ```
   
   1. SPP 방식이 속도가 빠름.

## Step 4. ADK Source 수정.
1. `Sink_audio_prompts.c`
   ```c
   #if 0 /* Jace. PSKEY_FSTAB 1000 파티션 mount 를 위해 panic() 무시 처리함. */
   if(!PartitionMountFilesystem(PARTITION_SERIAL_FLASH, theSink.audio_prompt_language , PARTITION_LOWER_PRIORITY))
        Panic();
   #else
   PartitionMountFilesystem(PARTITION_SERIAL_FLASH, theSink.audio_prompt_language , PARTITION_LOWER_PRIORITY);
   #endif
   ```

1. `Main.c` / `Sink_private.h` / `Sink_upgrade.c`
   > _Upgrade complete 후, Reboot 시 “전원이 켜집니다” VP 재생을 위해 해당 파티션을 마운트 시킨 후에는 Upgrade commit 시 이전 파티션을 삭제하지 못하여, 추후 재차 Upgrade 시 파티션 접근 에러 발생함. 이를 보완하기 위하여, complete 후 Reboot 시 “전원이 켜집니다” VP 재생하지 않도록 보완 코드 삽입._

## Step 5. PSKEY 변경.
1. `USR5` : 4~7번 파티션 비어있음, VP index 결정시 참조함.
   ```c
   &028F = 00F0 0000
   ```
   
1. `USR7` : 4개 국어, 64개 문장
   ```c
   &0291 = 0008 0010 0004 0009 000B 0005 0022 0040 0000 0000 0F0F 0000
   ```
   
1. `USR26` : 통화버튼 짧게, Select Audio Prompt Language Mode
   ```c
   &02A4 = 2b08 0000 6008 4708 0008 2008 4808 0010 2008 4902 0008 2008 4a09 0008 2008 4a0a 0008 2008 4a0c 0008 2008 4b02 0010 2008 4c09 0010 2008 4c0a 0010 2008 4c0c 0010 2008 5704 0008 2000 5804 0010 2000 4602 0000 6000 d108 000a 3fff d208 000c 3fff d008 000a 3fff d308 0100 3fff d408 0100 3fff d008 000c 3fff 0000 0000 0000 0000 0000 0000
   ```
   
1. `USR30` : 16개 VP 에 대한 이벤트
   ```c
   &02A8 = 4001 0000 3fff 4701 0001 3fff 4715 0002 3fff 470f 0003 3fff 470e 0004 3fff 470d 0005 3fff 4704 0006 3ffe 4003 0007 3fff 400b 0008 3fff 4009 0009 3fff 4008 000a 3fff 4717 000b 3fff 4070 000c 3fff 4071 000d 3fff 4705 000e 3fff 4002 000f 3fff
   ```
   
1. `FSTAB` : 우선순위에 따른 파티션 설정 (내부0파티션, 외부0파티션, 외부1, 외부2, 외부3)
   ```c
   &25E6 = 0000 1000 1001 1002 1003
   ```

## Step 6. 업그레이드 파일 생성.
1. 새로운 VP 데이터 생성.
   1. 업그레이드를 통해서는 파티션 구조 변경 불가능.
   
   1. 기존 VP 데이터에서 VP 교체 가능함. (또는 추가 PSKEY 변경을 통해 VP language 축소는 가능)  
   ☞ 파티션 단위 업그레이드 (Language 교체)
   
   1. [Step 1.-1.](https://github.com/luvinland/configuration-vp-using-ota-for-csr867x/blob/master/doc_source/configuration-vp-using-ota-for-csr867x.md#step-1-vp-%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%8C%8C%EC%9D%BC-idx-prm-%EC%83%9D%EC%84%B1) 참조하여 신규 VP 데이터 생성.  
   예.) `\01E` 폴더에 `\\header`, `\prompts` 이동. (`\01E` : 0~15 데이터)
   
   1. VP 데이터 폴더를 xuv 파일로 변경. (packing)
      ```c
      \tools\bin\packfile.exe 01E ptn01E.xuv
      ```
      
   1. Upgrade 를 위한 upd 파일 작성. `upgrade_partition.upd`
      ```c
      # ADK upgrade requires an empty signature appended to the end of the file. add_empty_signature
      
      # Set the upgrade version and previous version(s)
      # that are compatible to upgrade from. The minor
      # version can be '*' to act as a wildcard.
      upgrade_version 2.0 # Sink_upgrade.c 버전 정보 참조.
      compatible_upgrade 0.*
      compatible_upgrade 1.*
      compatible_upgrade 2.*
      
      # Set the ps config version and previous version(s)
      # that are compatible to upgrade from
      ps_config_version 2 # Sink_upgrade.c 버전 정보 참조.
      ps_prev_config_version 0
      ps_prev_config_version 1
      
      # list all partition starting from index 0 including partition type
      # <partition number> <partition type> <full path filename>
      # DFU file with file system
      0 3 ptn01E.xuv # 3번 타입의 0번 파티션을 ptn01E.xuv 파일로 업그레이드 함.
      ```
      
   1. 업그레이드 xuv 파일 생성.
      ```c
      \tools\bin\UpgradeFileGen.exe upgrade_partition.upd upgrade_file.xuv
      ```
      
   1. Xuv 파일을 업그레이드 bin 파일로 변환.
      ```c
      \tools\bin\XUV2BIN.exe -d upgrade_file.xuv upgrade_file.bin
      ```
      
   1. 생성된 bin 파일 단말기에 저장.

## Step 7. GAIA Control App. 이용한 Upgrade.
1. 헤드셋 연결 후 Upgrade 메뉴 진입.

1. 저장한 bin 파일 선택 후 업그레이드 진행.

1. `File transfer complete` [CONTINUE] → `Data commit` [CONTINUE] → `Upgrade complete` [OK] → `Power OFF` 됨 (“전원이 꺼집니다” VP 재생됨).

   ![03](https://user-images.githubusercontent.com/26864945/55312027-71f62200-549f-11e9-8dfe-c3cac2a8a082.PNG)

## Comment.
1. Upgrade 시 0번 파티션의 추가 파티션인 4번 파티션에 데이터가 저장됨. (Double 구성 [Step 2.-4.](https://github.com/luvinland/configuration-vp-using-ota-for-csr867x/blob/master/doc_source/configuration-vp-using-ota-for-csr867x.md#step-2-%EA%B8%B0%EB%B3%B8-%ED%8C%8C%ED%8B%B0%EC%85%98-%EC%83%9D%EC%84%B1) 참조.)

1. Upgrade complete 후, Reboot, Commit 시 0번 파티션 데이터 삭제함.

1. `PSKEY_FSTAB` 에 1000 은 1004 로 교체됨. (파티션 index 교체)
   ```c
   &25E6 = 0000 1004 1001 1002 1003
   ```
   
1. 추후, 0번 파티션 Upgrade 시 4번 파티션 데이터 삭제되며, `PSKEY_FSTAB` 의 1004 는 1000 으로 교체됨.
   ```c
   &25E6 = 0000 1000 1001 1002 1003
   ```
   
1. 예.) 3번 파티션 Upgrade 시 7번 파티션에 데이터 저장되며, 3번 파티션 삭제되고, `PSKEY_FSTAB` 의 1003 은 1007 로 교체됨.
   ```c
   &25E6 = 0000 1000 1001 1002 1007
   ```
   
1. ADK 4.0 기준으로 작성됨
