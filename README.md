
# 💻 ESP32-CHAT-COMMUNICATION-OVER-ESPNOW

<p align="center">
<img src="https://github.com/Jviraj/Blog/blob/main/assets/ESP32%20(2).png"/>
 </p>
 
<!-- ABOUT THE PROJECT -->
## 📃 About The Project

* Aim of the project is to build a chat system between two ESP32 development boards using ESPNOW.
* We will require two ESP32 devices.


## ✔️ Why ESPNOW?
* The chat system should be fast,secure and easy to use.
* Communication protocol used is ESP-NOW as
  * It overcomes the drawbacks of traditonal wifi connection.
  * The pairing between devices is needed prior to their communication. After the pairing is done, the connection is secure and peer-to-peer.
  * ESP NOW does not require a router for the connection,Thus this project can be used anywhere,at any remote places.
  * If suddenly one of the boards loses power or resets, when it restarts, it will automatically connect to its peers.


## 📋 File Structure
     .
    ├── Components              # Contains files of specific library of functions or Hardware used
    │    ├──esp_now             # Contains the code to setup connection using ESP-NOW. 
    │    ├──CMakeLists.txt      # To include this component in a esp-idf 
    ├── docs                    # Documentation files 
    │   ├── report.pdf          # Project report
    │   └── results             # Folder containing the video, gifs of the result
    ├── main                    # Source files of project
    │   ├──main.c               # Main Source code.
    │   ├──kconfig.projbuild    # Shows the menu of project configuration
    │   ├──CMakeLists.txt       # To include source code files in esp-idf.
    ├── CmakeLists.txt          # To include components and main folder while executing
    ├── LICENSE
    └── README.md 


## Getting Started
### Prerequisites
* **ESP-IDF v4.0 and above**
### Installation
Clone the repo
```sh
git clone https://github.com/RISHI27-dot/ESP32-chat-communication-over-wifi
```
<!-- USAGE EXAMPLES -->
## Usage
### Configuration

```
idf.py menuconfig
```
* `Chat history`
  * `Store command history in flash` - to store the chat history ans use previous chats.
  
* `ESP-NOW Configuration`
  * `Send len` - espnow packet length.

<p align="center">
<img src="https://github.com/Jviraj/Blog/blob/main/assets/menuconfig.png"/>
 </p>
 
### Build
```
idf.py build
```
### Flash and Monitor
* Connect two esp32 through ports and run the following command on two seprate terminals.
* The terminals will act as user interface. 
```
idf.py -p /dev/ttyUSB0 flash monitor

```
```
idf.py -p /dev/ttyUSB1 flash monitor

```


## 📄 Understanding Code
### Base Mac Address
[Code](https://github.com/espressif/esp-idf/tree/master/examples/system/base_mac_address/main)

On flashing the code, the console shows the derived MAC address of each network interface as such 

<p align="center">
<img src="https://github.com/Jviraj/Blog/blob/main/assets/mac_add.png"/>
 </p>
 
### Console
ESP-IDF provides console component, which includes building blocks needed to develop an interactive console over serial port.This component is used in this project to create a frontend chat interface. It also includes followint properties:
- Line editing, provided by linenoise library.
- Scrolling through chat history. 
- Accessing previous chats.
- Tab auto-completion.


In `app_main()`, we initialize nvs, filesyste and the console.A task is created to run the console and `task_console` is passed as the first argument. It is a	pointer to the task entry function that will execute the task.

```cpp
void app_main(void)
{
    initialize_nvs();
#if CONFIG_STORE_HISTORY
    initialize_filesystem();
    ESP_LOGI(TAG, "Command history enabled");
#else
    ESP_LOGI(TAG, "Command history disabled");
#endif
    /*to initialize the console in default configurarion to take user input for terminal*/
    initialize_console();
    /*setup espnow interconnection*/
    espnowinit();
 
    printf("\nChat Communication\n");//start of the chat communication

    /* Figure out if the terminal supports escape sequences */
    /*this part is to take the input of strings in proper format*/
    int probe_status = linenoiseProbe();
    if (probe_status)
    { /* zero indicates success */
        linenoiseSetDumbMode(1);
#if CONFIG_LOG_COLORS
        /* Since the terminal doesn't support escape sequences,
         * don't use color codes in the prompt.
         */
        prompt = PROMPT_STR "> ";
#endif
    }
    /*creation of the console task*/
    xTaskCreate(task_console, "task_console", 3000, NULL, 3, &console);
} 
```
The `task_console()` function.
```cpp

void task_console()
{
    console_to_espnow_send = xQueueCreate(10, 250 * sizeof(char));
    while (1)
    {
        /* Get a line using linenoise.
         * The line is returned when ENTER is pressed.
         *if the user enters a null message the prompt again for message
         */
        /*take user input typed at the prompt*/
        char *line = linenoise(prompt);
        /*prompt again if the string is empty*/
        while (line == NULL) { /* Break on EOF or error */
            ESP_LOGW(TAG, "Enter a message!!");
            line = linenoise(prompt);
        }
        /* Add the command to the history if not empty*/
        if (strlen(line) > 0)
        {
            linenoiseHistoryAdd(line);
#if CONFIG_STORE_HISTORY
            /* Save command history to filesystem */
            linenoiseHistorySave(HISTORY_PATH);
#endif
            if (xQueueSend(console_to_espnow_send, &line, portMAX_DELAY) != pdTRUE)
            {
                ESP_LOGW(TAG, "Send data to esp now failed.........");
            }
            espnow_start();
        }
        /* linenoise allocates line buffer on the heap, so need to free it */
        linenoiseFree(line);
        vTaskSuspend(console);
    }

}
```
### ESPNOW

ESP-NOW is a protocol developed by Espressif, which enables multiple devices to communicate with one another without using Wi-Fi. The protocol is similar to the low-power 2.4GHz wireless connectivity that is often deployed in wireless mouses. So, the pairing between devices is needed prior to their communication. After the pairing is done, the connection is secure and peer-to-peer, with no handshake being required.

The function `espnowinit()` gets called in app main, which makes the setup for espnow interconnection and to add peers to establish connection between them.

```cpp
void espnowinit(void)
{
    uint8_t my_mac[6];
    esp_efuse_mac_get_default(my_mac);
    char my_mac_str[13];
    ESP_LOGI(TAG, "My mac %s", mac_to_str(my_mac_str, my_mac));
    bool is_current_esp1 = memcmp(my_mac, esp_1, 6) == 0;
    uint8_t *peer_mac = is_current_esp1 ? esp_2 : esp_1;
    //initialize wifi in default configuration
    nvs_flash_init();
    tcpip_adapter_init();
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
    //initialize espnow 
    ESP_ERROR_CHECK(esp_now_init());
    //register send and recive callback function
    ESP_ERROR_CHECK(esp_now_register_send_cb(on_sent));
    ESP_ERROR_CHECK(esp_now_register_recv_cb(on_receive));
    //setup the peer list 
    esp_now_peer_info_t peer;
    memset(&peer, 0, sizeof(esp_now_peer_info_t));
    memcpy(peer.peer_addr, peer_mac, 6);
    esp_now_add_peer(&peer);
}
```

This task is utilized to send data. The function used to send data is `esp_now_send()`.
```cpp
static void espnow_task_send(void)
{
    ESP_ERROR_CHECK(esp_now_send(NULL, (uint8_t *)send_buffer, strlen(send_buffer)));
    vTaskDelay(pdMS_TO_TICKS(1000));

    vTaskDelete(send);
}
```

The callback function `on_sent()` is called when data is sent from one ESP32. The `console()` task resumes when the data is sent.
```cpp
void on_sent(const uint8_t *mac_addr, esp_now_send_status_t status)
{
    vTaskResume(console);
}
```

The callback function `on_receive` is called when data is received. And the chat data is displayed on the console. It is done in a task which is created in this function. The function espnow_task_recv() executes this task.
```cpp
void on_receive(const uint8_t *mac_addr, const uint8_t *data, int len)
{
    data_g = data;
    len_g = len;
    xTaskCreate(espnow_task_recv, "espnow_task_recv", 5000, NULL, 5, &recv);
}
```

The task created previously with `espnow_task_recv()` as its execution function has a higher priority than the console task so it interrupts the task console to display the received data.
```cpp
void espnow_task_recv(void)
{
    if (len_g != 0)
    {
        printf("\nReceived: %.*s\n", len_g, data_g);
    }
    vTaskDelete(recv);
}
```
Once the chat data is displayed on the console, the user is prompted again for the next chat and the cycle repeats.

## 🧭 Flowchart

<p align="center">
<img src="https://github.com/Jviraj/Blog/blob/main/assets/code_flow.png"/>
 </p>
 
## Result
![**result gif**](https://github.com/Jviraj/Blog/blob/main/assets/result.gif)

## Conclusion
* The maximum length of data that can be sent is 250 bytes.
* The mac address of the ESP32, to which the data is sent, is required in this example. The code can be modified such that we don't require it.
* Encryption of data is yet to be added.
* The range of `ESPNOW` is about 500m.

## Contributors
* [Viraj Jagdale](https://github.com/Jviraj)
* [Rishikesh Donadkar](https://github.com/RISHI27-dot)

## Git repository
[ESP32-Chat-Communication-Over-wifi](https://github.com/RISHI27-dot/ESP32-chat-communication-over-wifi)
