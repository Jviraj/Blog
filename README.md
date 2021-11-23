# ESP-32 CHAT COMMUNICATION
## 📖TABLE OF CONTENTS
* [Technologies Used](#technologies-used)
* [File Structure](#file-structure)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
### Technologies Used
<!-- GETTING STARTED -->
### Getting Started

### Prerequisites

* **ESP-IDF v4.0 and above**

  You can visit the [ESP-IDF Programmming Guide](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html#installation-step-by-step) for the installation steps.

### Installation
Clone the repo
```sh
git clone https://github.com/RISHI27-dot/ESP32-chat-communication-over-wifi
```
### File Structure
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
