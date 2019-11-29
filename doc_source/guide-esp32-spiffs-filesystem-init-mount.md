# Get Started
ESP32 SPIFFS init & mount 가이드.

## Step 1. SPIFFS partition 생성
1. 파티션 파일 `partitions_example.csv` 작성.
	```c
	# Name,   Type, SubType, Offset,  Size, Flags
	# Note: if you change the phy_init or app partition offset, make sure to change the offset in Kconfig.projbuild
	nvs,      data, nvs,     0x9000,  0x6000,
	phy_init, data, phy,     0xf000,  0x1000,
	factory,  app,  factory, 0x10000, 1M,
	storage,  data, spiffs,  ,        0xF0000, // 여기서는 0x110000 에서 부터 0xF0000 사이즈만큼 설정
	```

1. menuconfig 파티션 설정.
	1. menuconfig → Partition Table → Partition Table → `Custom partition table CSV` 선택
		![image](https://user-images.githubusercontent.com/26864945/69838027-fd28f100-1294-11ea-9046-311f61a039ef.png)

	1. `Custom partition CSV file` 지정
		![image](https://user-images.githubusercontent.com/26864945/69838192-d1f2d180-1295-11ea-8b89-c314919a4452.png)

	1. Save → Exit

## Step 2.파티션 초기화 및 마운트.
1. IDF 의 SPIFFS components 사용.
	```c
	#include "esp_spiffs.h"
	```
	> _esp-va-sdk 의 경우, IDF_PATH base 임._

1. spiffs init & mount
	```c
	esp_err_t ctc_spiffs_init(void)
	{
		ESP_LOGI(TAG, "Initializing SPIFFS");

		esp_vfs_spiffs_conf_t conf = {
			.base_path = "/spiffs",
			.partition_label = NULL,
			.max_files = 5,
			.format_if_mount_failed = true
		};

		// Note: esp_vfs_spiffs_register is an all-in-one convenience function.
		esp_err_t ret = esp_vfs_spiffs_register(&conf);

		if (ret != ESP_OK) {
			if (ret == ESP_FAIL) {
				ESP_LOGE(TAG, "Failed to mount or format filesystem");
			} else if (ret == ESP_ERR_NOT_FOUND) {
				ESP_LOGE(TAG, "Failed to find SPIFFS partition");
			} else {
				ESP_LOGE(TAG, "Failed to initialize SPIFFS (%s)", esp_err_to_name(ret));
			}
		}

		size_t total = 0, used = 0;
		ret = esp_spiffs_info(NULL, &total, &used);
		if (ret != ESP_OK) {
			ESP_LOGE(TAG, "Failed to get SPIFFS partition information (%s)", esp_err_to_name(ret));
		} else {
			ESP_LOGI(TAG, "Partition size: total: %d, used: %d", total, used);
		}

		return ret;
	}
	```

1. Success  
	![image](https://user-images.githubusercontent.com/26864945/69845099-acc08c00-12b2-11ea-8f0f-9282fe64e78d.png)
