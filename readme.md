## PHẦN 01 - Giới Thiệu

Zephyr OS là một RTOS mã nguồn mở được quản lý bởi Linux Foundation

Zephyr cung cấp

```cpp
Kernel RTOS
Scheduler
Driver framework
Device model
Networking stack
Bluetooth
USB
File system
Build system
...
```

### Sẽ ra sao nếu chúng ta phát triển STM32, ESP32,...

ESP-IDF cho ESP32

```cpp
gpio_set_level(GPIO_NUM_2, 1);
```

STM32HAL cho STM32

```cpp
HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
```

Nếu dùng Zephyr nó giống nhau hết trên tất cả các board

```cpp
gpio_pin_set_dt(&led, 1);
```

## PHẦN 02 - Setup Môi Trường

### Bước 01 - Install các tool cần thiết

```cpp
winget install Kitware.CMake Ninja-build.Ninja oss-winget.gperf Python.Python.3.12 Git.Git oss-winget.dtc wget 7zip.7zip
```

| Công cụ | Chi tiết                                    |
| ------- | ------------------------------------------- |
| CMake   | Dùng CMake để quản lý trình biên dịch       |
| Ninja   | Build backend thay thế Make                 |
| Python  | Min ver 3.12.0                              |
| Git     | Dùng để clone các repo                      |
| Gperf   | Tạo hash table > giúp tìm config nhanh O(1) |
| Dtc     | Device tree compiler                        |
| Wget    | Download file với cmd (toolchain, sdk,...)  |
| 7-Zip   | Giải nén file                               |

### Bước 02 - Tạo workspace

Bước này là workspace của chính mọi người, mọi người thích ở đâu thì tạo ở đó nhé, kể cả desktop cũng được nếu ổ C mọi người nhiều 😊

```cpp
mkdir E:\zephyr
cd E:\zephyr
mkdir zephyrproject
cd zephyrproject
```

### Bước 03 - Setup venv (virtual environment)

```cpp
py -3.12 -m venv .venv
```

Lệnh này tạo một Python virtual environment (tạo file .venv)

#### 3.1 Venv là gì?

Là một bản python riêng biệt, cách ly với python hệ thống
1 venv bao gồm

- Python interpreter riêng
- PIP riêng
- Packages riêng

#### 3.2 Tại sao cần?

Tránh xung đột ver. Ví dụ
Không có venv

```cpp
Project A cần abc v1.0
Project B cần abc v2.0
Python hệ thống cài abc v2.0
--> Project A die 👌
```

Có venv

```
Project A cần abc v1.0
Project B cần abc v2.0
Python venvA cài abc v1.0
Python venvB cài abc v2.0
--> Chạy ngon ơ cả 2 project 😊
```

#### 3.3 Activate và Deactive

```cpp
.venv\Scripts\activate : giúp activate
deactivate: giúp deactivate
```

### Bước 04 - Tải các packages cần thiết

Đầu tiên là tải west, init và update

```cpp
pip install west
west init .
west update
```

#### 4.1 west là gì?

west là thằng giúp quản lý toàn bọ workflow của zephyr
Những thứ nó có thể làm:

- west build
- west flash
- ...

#### 4.2 west init và update

- Lệnh west init: bản chất nó sẽ clone zephyr main repo
- Lệnh west update: Lúc này nó sẽ tải cả đống dependencies về. Về cơ bản nó sẽ đọc file west.yml và clone tất cả các module cần thiết về (VD: HAL, Libraries, tools,...)

#### 4.3 west zephyr-export

```cpp
west zephyr-export
```

Lệnh này giúp xuất zephyr package ra cho Cmake. Hiểu đơn giản là nó đăng ký zephyr như một CMake package để các project bên ngoài có thể tìm thấy Zephyr một cách dễ dàng

Tại sao phải làm như vậy?
Hiểu đơn giản như sau:

- zephyrproject chỉ là workspace để bạn clone nguyên cái zephyr về
- application space: Đây mới là nơi bạn lưu src của dự án

Sau khi bạn chạy lệnh kia xong thì dùng project bạn lưu ở đâu thì nó vẫn tìm được Zephyr vì Zephyr đã được đăng kí đường dẫn vào CMake registry

#### 4.4 Cài python dependencies

```cpp
west packages pip --install
```

Lệnh này cài hàng tỉ tỉ package python mà Zephyr cần - lâu lắm 🤣 - cố gắng chờ ✌️

### Bước 05 - Tải Zephyr SDK

```cpp
west sdk install
```

Lệnh trên sẽ tải full SDK về

#### 5.1 SDK là gì?

Software development kit - bộ công cụ phát triển phần mềm
SDK:

- Compiler (VD: arm-zephyr-ebai-gcc): Compile cho STM32
- GBD (VD: arm-zephyr-ebai-gdb): Debugger
- ...

Thằng này tải lúc lâu, ae cố gắng chờ 🤣

#### 5.2 List sdk, board

Chúng ta có thể list ra xem nó có sử dụng được cho dự án của mình không

```cpp
west sdk list: List full ra sdk cho mình nhìn
west boards | findstr esp32: Xem nó hỗ trợ những board nào
```

## PHẦN 03 - Build Dự Án Đầu Tiên
