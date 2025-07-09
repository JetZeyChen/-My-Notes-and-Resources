# 程序MAIN函数的流程

```
void app_main(void)
{
    // Initialize the default event loop
    ESP_ERROR_CHECK(esp_event_loop_create_default());

    // Initialize NVS flash for WiFi configuration
    esp_err_t ret = nvs_flash_init();
    if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
        ESP_LOGW(TAG, "Erasing NVS flash to fix corruption");
        ESP_ERROR_CHECK(nvs_flash_erase());
        ret = nvs_flash_init();
    }
    ESP_ERROR_CHECK(ret);

    // Launch the application
    Application::GetInstance().Start();
}

```

步骤：1.初始化默认事件-->2.为WIFI配置初始化NVS Flash(flash为外部还是内部存疑?需要查看硬件原理图)-->3.开始应用程序

## Start()函数

### 各种外设功能初始化

1. 初始化开发板实例

   ```
       auto &board = Board::GetInstance();
       SetDeviceState(kDeviceStateStarting);
   ```

2. 初始化显示功能

   ```
   auto display = board.GetDisplay();
   ```

3. 初始化音频编解码器

   ```
   auto codec = board.GetAudioCodec();
   ... ...
   ```

4. 创建音频任务

   ```
       xTaskCreatePinnedToCore([](void *arg){
           Application* app = (Application*)arg;
          	app->AudioLoop();
           vTaskDelete(NULL); }, "audio_loop", 4096 * 2, this, 8, 		 			&audio_loop_task_handle_, realtime_chat_enabled_ ? 1 : 0);
   ```

5. 建立网络连接

   ```
   board.StartNetwork();
   ```

6. 检查固件版本或者获取MQTT Broker 地址

   ```
   CheckNewVersion();
   ```

7. 初始化协议（Websocket/**MQTT**(本程序使用**MQTT**协议)）

   ```
   protocol_ = std::make_unique<MqttProtocol>();
   ... ...
   ```

8. 等待固件版本确认完毕，生成提示音

   ```
       // Wait for the new version check to finish
       xEventGroupWaitBits(event_group_, CHECK_NEW_VERSION_DONE_EVENT, pdTRUE, pdFALSE, portMAX_DELAY);
       SetDeviceState(kDeviceStateIdle);
       std::string message = std::string(Lang::Strings::VERSION) + ota_.GetCurrentVersion();
       display->ShowNotification(message.c_str());
       display->SetChatMessage("system", "");
       // Play the success sound to indicate the device is ready
       ResetDecoder();
       PlaySound(Lang::Sounds::P3_SUCCESS);
   ```

9. 进入主任务

   ```
       // Enter the main event loop
       MainEventLoop();
   ```

   

# MQTT协议实现过程

## MQTT 发送端接入云平台流程



# WIFI相关配置

## 接入WiFi过程

### board.StartNetwork()函数解析

1. 在启动的时候用户可以按下启动键进入WIFI配置模式

   >```
   >    if (wifi_config_mode_) {
   >        EnterWifiConfigMode();
   >        return;
   >    }
   >```
   >
   >```wifi_config_mode_```为标志位变量，标志WiFi配置模式

2. 从SSID管理器（其从NVS Flash中读取）中获取SSID历史列表，如果列表为空则需要进入WiFi配置模式

   >```
   >    if (ssid_list.empty()) {
   >        wifi_config_mode_ = true;
   >        EnterWifiConfigMode();
   >        return;
   >    }
   >```

3. 如果SSID列表有历史记录，则无需配置

   >WIFI配置函数```EnterWifiConfigMode()```如下：
   >
   >```
   >void WifiBoard::EnterWifiConfigMode() {
   >    auto& application = Application::GetInstance();
   >    application.SetDeviceState(kDeviceStateWifiConfiguring);
   >
   >    auto& wifi_ap = WifiConfigurationAp::GetInstance();
   >    wifi_ap.SetLanguage(Lang::CODE);
   >    wifi_ap.SetSsidPrefix("Xiaozhi");
   >    wifi_ap.Start();
   >
   >    // 显示 WiFi 配置 AP 的 SSID 和 Web 服务器 URL
   >    std::string hint = Lang::Strings::CONNECT_TO_HOTSPOT;
   >    hint += wifi_ap.GetSsid();
   >    hint += Lang::Strings::ACCESS_VIA_BROWSER;
   >    hint += wifi_ap.GetWebServerUrl();
   >    hint += "\n\n";
   >    
   >    // 播报配置 WiFi 的提示
   >    application.Alert(Lang::Strings::WIFI_CONFIG_MODE, hint.c_str(), "", Lang::Sounds::P3_WIFICONFIG);
   >    
   >    // Wait forever until reset after configuration
   >    while (true) {
   >        int free_sram = heap_caps_get_free_size(MALLOC_CAP_INTERNAL);
   >        int min_free_sram = heap_caps_get_minimum_free_size(MALLOC_CAP_INTERNAL);
   >        ESP_LOGI(TAG, "Free internal: %u minimal internal: %u", free_sram, min_free_sram);
   >        vTaskDelay(pdMS_TO_TICKS(10000));
   >    }
   >}
   >```
   >
   >其中将ESP32S3作为WiFi接入点，```  wifi_ap.SetSsidPrefix("Xiaozhi");```得知其SSID前缀为“Xiaozhi”
   >
   >在```void WifiConfigurationAp::StartAccessPoint()```函数中可以看到其调用```std::string ssid = GetSsid();```语句生成以Xiaozhi为前缀+MAC地址随机生成的SSID，且密码为无。

4. 启用wifi_station.Start()函数调用底层库进行WiFi连接

   >```
   >```
   >
   >
   >
   >

## 本地存储WiFi列表及WIFI配置

