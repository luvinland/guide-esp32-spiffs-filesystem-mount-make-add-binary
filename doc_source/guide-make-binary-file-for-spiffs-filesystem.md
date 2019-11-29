# Get Started
Binary file (image file) 생성 및 사용 가이드.

## Step 1. SPIFFS Bin 파일 만들기
1. esp-va-sdk 에 git 에 포함된 win32 용 mkspiffs.exe 사용.
	> _git 에서 배포하는 spiffs ([github/spiffs](https://github.com/igrr/mkspiffs)) 가 있으며, Espressif 에서 배포하는 [windows 용 toolchain](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/windows-setup.html#toolchain-setup) 에서는 build 실패함. linux 환경에서는 문제 없음._

1. 실행 파일이 존재하는 폴더 내 임시 폴더 생성 → 저장하고자 하는 파일 넣기 → 이미지 파일 (Bin) 생성.
	```c
	mkspiffs.exe -c .\wmfw -b 4096 -p 256 -s 0xf0000 .\wmfw\SCSH_COOKE_20190826.bin
	```
	![image](https://user-images.githubusercontent.com/26864945/69849547-62dea280-12c0-11ea-9102-50635c85277f.png)

## Step 2. SPIFFS 파티션에 이미지 파일(*.bin) Writing.
1. esptool.py 명령어 이용하여 flash write.
	```c
	esptool.py --chip esp32 --port COM7 --baud 115200 write_flash -z 0x110000 SCSH_COOKE_20190826.bin
	```
	![image](https://user-images.githubusercontent.com/26864945/69848923-b5b75a80-12be-11ea-94ef-98a21ef8eb9f.png)

1. make monitor 에서 원본 파일 SCSH_COOKE_20190826.wmfw 이 SPIFFS 파티션에 저장되어 있음을 확인할 수 있다.
	![image](https://user-images.githubusercontent.com/26864945/69849652-adf8b580-12c0-11ea-8221-5f48d80af527.png)
	
## Step 3. 파일 접근
1. 아래 예시 코드 참고.
	```c
	ESP_LOGI(TAG, "[APP] Free memory: %d bytes", esp_get_free_heap_size());

	esp_err_t ret;

	const char *filename = "/spiffs/SCSH_COOKE_20190826.wmfw";

	// Use settings defined above to initialize and mount SPIFFS filesystem.
	ctc_spiffs_init();
	// Choose which ADSP2 core to download to
	selectCore();

	// WMFW file
	ret = (esp_err_t)ProcessWMFWFile(filename);
	if (ret != ESP_OK) {
		ESP_LOGE(TAG, "[ 0 ] process wmfw file error : %d", ret);
	}
	else {
		ESP_LOGE(TAG, "[ 0 ] process wmfw file success");
	}
	```
## Comment.
1. 위 예시에서는 하나의 원본 파일만 사용하였으나, 여러 개의 파일 저장 가능함.
1. 하나의 이미지 파일(Bin) 으로 생성하여, 코드 상에서 원본 파일명으로 접근.
