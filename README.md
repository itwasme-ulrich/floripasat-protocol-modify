# floripasat-protocol-modify
Giao thức truyền gói tin giữa các board, tham khảo từ Floripasat
# Welcome to the floripasat-protocol-modify wiki!
# Định nghĩa

## Gói tin - Khung truyền
| Trường | Loại | Độ dài (bytes) |
|--|--|--|
| SOD | Start of Data (0xCA) | 1 |
| SRC | Source Address (Phát) | 1 |
| DST | Destination address (Thu) | 1 |
| LEN | Length of the packet payload (*) | 1 |
| TYPE | Type of packet/frame | 1 |
| Payload| Data of the packet/frame | n (**) |
| CRC | CRC-16/XMODEM | 2 |
| EOF | End of Frame (0xEF) | 1 |

(*) Độ dài Payload không bao gồm Header(SOD,SRC,DST), FOOTER, và các byte ESC.
(**) Payload có giá trị độ dài nằm trong khoảng 0 - 247 bytes. Bởi vậy, độ dài 1 frame có giá trị từ 8 - 255 bytes

## Địa chỉ đích
Hiện tại trong file `fsp.h` được định nghĩa sẵn:
```c
#define  FSP_ADR_COPC 	1 /**< COPC module address. */
#define  FSP_ADR_PMU 	2 /**< PMU module address. */
#define  FSP_ADR_PDU 	3 /**< PDU module address. */
#define  FSP_ADR_CAM 	4 /**< CAM module address. */
#define  FSP_ADR_IOU 	5 /**< IOU module address. */
```
|  Module| Địa chỉ |
|--|--|
| CPOC [Master] | 1 |
| PMU | 2 |
| PDU | 3 |
| CAM | 4 |
| IOU | 5 |

## Byte định nghĩa, Byte chuyển đổi, Byte chèn

![image](https://github.com/itwasme-ulrich/floripasat-protocol-modify/assets/110149749/4037fab1-cb88-4cf4-a94f-d6b7a0dd5ca6)

| Byte| Định nghĩa| Giá trị |
|--|--|--|
| SOD | Bắt đầu của một Frame | 0xCA |
| ESC | Byte Escape | 0xBD |
| EOF | Kết thúc một Frame | 0xEF |
| TSOD| Byte chuyển đổi của SOD | 0xDC |
| TESC| Byte chuyển đổi của ESC | 0xDB |
| TEOF| Byte chuyển đổi của EOF | 0xDE |

Các Byte SOD, ESC, EOF nằm trong gói tin Raw sẽ được Encode, thay thế bằng cặp  byte ESC+Txxx tương ứng. 
Nhờ vậy, không có Byte nào SOD và EOF nằm trong dữ liệu, đảm bảo độ sạch 1 Frame.

## Packet Types - Kiểu gói tin
```c
// Types of packets

#define  FSP_PKT_TYPE_DATA 				1
#define  FSP_PKT_TYPE_DATA_WITH_ACK 			2
#define  FSP_PKT_TYPE_CMD 				3
#define  FSP_PKT_TYPE_CMD_WITH_ACK 			4
#define  FSP_PKT_TYPE_ACK 				5 
#define  FSP_PKT_TYPE_NACK 				6
#define  FSP_PKT_TYPE_CMD_W_DATA 			7
#define  FSP_PKT_TYPE_CMD_W_DATA_ACK 			8
```
| Type| Định nghĩa| Quy ước |
|--|--|--|
| DATA | Gói tin chỉ mang thông tin là Payload | 1 |
| DATA WITH ACK | Gói tin mang thông tin là Payload, yêu cầu bên Decode trả lời ACK | 2 |
| CMD | Gói tin chỉ mang thông tin là Mã lệnh (Payload[0]) | 3 |
| CMD WITH ACK | Gói tin mang thông tin là Mã lệnh (Payload[0]), yêu cầu bên Decode trả lời ACK | 4 |
| ACK | Gói tin ACK, không mang Payload |5 |
| NACK | Gói tin NACK, không mang Payload  | 6 |
| CMD with DATA| Gói tin mang thông tin là Mã lệnh (Payload[0]) kèm dữ liệu (Payload[++])  | 7 |
| CMD with DATA - ACK | Gói tin mang thông tin là Mã lệnh (Payload[0]) kèm dữ liệu (Payload[++]), yêu cầu bên Decode trả lời ACK | 8 |

## Lệnh - Mã lệnh
Các lệnh ví dụ được định nghĩa sẵn
```c
#define  FSP_CMD_HEART_BEAT 						1 
#define  FSP_CMD_PDU_SINGLE_POWER_CONTROL 				2
#define  FSP_CMD_PDU_SINGLE_POWER_STATUS_REQUEST 			3 
#define  FSP_CMD_HIBERNATION 						4 
#define  FSP_CMD_RESET_CHARGE 						5
```
## CRC16
Kiểu kiểm tra được sử dụng là: CRC-16/XMODEM
Việc tính toán độ dài, CRC sẽ được tính toán trước khi Encode. Có nghĩa là các dữ liệu này sẽ nằm trong Raw Frame, nếu có giá trị ESC, SOD, EOF, việc encode sẽ xử lý.
## Mã lỗi - Error Return Code
```c
#define  FSP_PKT_NOT_READY 			0 /**< Packet not ready. */
#define  FSP_PKT_READY 				1 /**< Packet ready. */
#define  FSP_PKT_INVALID 			2 /**< Packet invalid. */
#define  FSP_PKT_WRONG_ADR 			3 /**< Packet with wrong address. */
#define  FSP_PKT_ERROR 				4 /**< Packet with error. */
#define  FSP_PKT_CRC_FAIL 			5 /**< Packet check CRC Fail. */
#define  FSP_PKT_WRONG_LENGTH 			6 /**< Packet wrong length. */
```
Sử dụng bằng việc ánh xạ bảng:
```c
const  char  *decode_error_msgs[7]  =  {
"Packet ready",
"Packet not ready",
"Packet invalid",
"Packet wrong address",
"Packet error",
"Packet CRC failed",
"Packet with wrong length"
};
```
# Sử dụng - Mã hoá
## Khởi động gói tin cho một thiết bị
```c
void  _init(void){
	fsp_init(FSP_ADR_CPOC);
}
```
Khởi động thiết bị sử dụng hàm `fsp_init($Thiết bị sử dụng);`

Định nghĩa thiết bị nhận:
```c
#define  DEST_ADDR FSP_ADR_IOU
// define  DEST_ADDR $Thiết bị đích
```

## Tạo một gói tin
Các gói tin có thể khởi tạo:
* ***Gói tin chỉ chứa dữ liệu***:
```c
void  fsp_gen_data_pkt(uint8_t  *data,  uint8_t  data_len,  uint8_t  dst_adr,  uint8_t  ack,  fsp_packet_t  *fsp);
```
Ví dụ sử dụng:
```c
uint8_t  payload[2];
payload[0]  =  channel;
payload[1]  =  sensor;
fsp_packet_t  fsp_pkt;
fsp_gen_data_pkt(payload,  sizeof(payload), DEST_ADDR, FSP_PKT_WITHOUT_ACK,  &fsp_pkt);
```
* ***Gói tin chứa dữ liệu - yêu cầu ACK***:
Truyền vào tham số: `FSP_PKT_WITH_ACK`
```c
fsp_gen_data_pkt(payload,  sizeof(payload), DEST_ADDR, FSP_PKT_WITH_ACK,  &fsp_pkt);
```
* ***Gói tin  chỉ chứa mã lệnh***:
```c
void  fsp_gen_cmd_pkt(uint8_t  cmd,  uint8_t  dst_adr,  uint8_t  ack,  fsp_packet_t  *fsp);
```
Ví dụ sử dụng:
```c
uint8_t  cmd  = FSP_CMD_PDU_SINGLE_POWER_CONTROL;
fsp_packet_t  fsp_pkt;
fsp_gen_cmd_pkt(cmd, DEST_ADDR, FSP_PKT_WITHOUT_ACK,  &fsp_pkt);
```
* ***Gói tin chứa mã lệnh - yêu cầu ACK***:
Truyền vào tham số: `FSP_PKT_WITH_ACK`
```c
fsp_gen_cmd_pkt(cmd, DEST_ADDR, FSP_PKT_WITH_ACK,  &fsp_pkt);
```
* ***Gói tin ACK***:
```c
void  fsp_gen_ack_pkt(uint8_t  dst_adr,  fsp_packet_t  *fsp);
```
Ví dụ sử dụng:
```c
fsp_packet_t  fsp_pkt;
fsp_gen_ack_pkt(DEST_ADDR,  &fsp_pkt);
```
* ***Gói tin NACK***:
```c
void  fsp_gen_nack_pkt(uint8_t  dst_adr,  fsp_packet_t  *fsp);
```
Ví dụ sử dụng:
```c
fsp_packet_t  fsp_pkt;
fsp_gen_nack_pkt(DEST_ADDR,  &fsp_pkt);
```
* ***Gói tin chứa mã lệnh - dữ liệu***:
```c
void  fsp_gen_cmd_w_data_pkt(uint8_t  cmd,  uint8_t  *data,  uint8_t  data_len,  uint8_t  dst_adr,  uint8_t  ack,  fsp_packet_t  *fsp);
```
Ví dụ sử dụng:
```c
uint8_t  cmd  = FSP_CMD_PDU_SINGLE_POWER_CONTROL;
uint8_t  payload[2];
payload[0]  =  channel;
payload[1]  =  sensor;
fsp_packet_t  fsp_pkt;
fsp_gen_cmd_w_data_pkt(cmd,  payload,  sizeof(payload), DEST_ADDR, FSP_PKT_WITHOUT_ACK,  &fsp_pkt);
```
* ***Gói tin chứa mã lệnh - dữ liệu - yêu cầu ACK***:
Truyền vào tham số: `FSP_PKT_WITH_ACK`
```c
fsp_gen_cmd_w_data_pkt(cmd,  payload,  sizeof(payload), DEST_ADDR, FSP_PKT_WITH_ACK,  &fsp_pkt);
```

## Mã hoá một gói tin
Gọi hàm:
```c
void  frame_encode(fsp_packet_t  *fsp,  uint8_t  *frame,  uint8_t  *frame_len);
```
Hàm này nhận vào địa chỉ tới gói tin, trả về mảng chứa các byte của Frame và độ dài Frame.
Ví dụ sử dụng:
```c
uint8_t  encoded_frame[FSP_PKT_MAX_LENGTH];
uint8_t  frame_len;
frame_encode(&fsp_pkt,  encoded_frame,  &frame_len);
```

## Gửi một gói tin:
Gửi từng Byte ra cổng giao tiếp:
```c
for  (int  i  =  0;  i  <  frame_len;  i++)  {
Uart_write(USART6,  encoded_frame[i]);
}
```

# Sử dụng - Giải mã
## Giải mã
Giải mã một Frame.
Một Frame luôn bắt đầu từ Byte SOD, kết thúc bằng byte EOF.
Sử dụng hàm:
```c
int  frame_decode(uint8_t  *buffer,  uint8_t  length,  fsp_packet_t  *pkt);
```
Hàm này nhận vào Buffer chứa dữ liệu và  chiều dài không bao gồm Byte SOD và Byte EOF.
Trả về một packet chứa cấu trúc đã giải mã`fsp_packet_t  fsp_pkt;` ,kèm theo mã lỗi.

```c
if  (!receiving)  {
	if  (rxData  == FSP_PKT_SOD)  {
		receiving  =  1;
		receive_index =  0;
		}
	}  else  {
		if  (rxData  == FSP_PKT_EOF)  {
			receiving  =  0;
			fsp_packet_t  fsp_pkt;
			int  ret  =  frame_decode((uint8_t  *)receive_buffer, receive_index,  &fsp_pkt);
			if  (ret  >  0)  {
				char  error_msg[50];
				sprintf(error_msg,  "Error: %s\r\n", decode_error_msgs[ret]);
				Uart_sendstring(USART1,  error_msg);
			}
		}else{
			receive_buffer[receive_index++]  =  rxData;
		}
		if  (receive_index >= FSP_PKT_MAX_LENGTH)  {
			// Frame quá  dài, reset lại
			receiving  =  0;
		}
}
```
Trong thư viện file `fsp.c`, có hàm 
```c
int  frame_processing(fsp_packet_t  *fsp_pkt);
```
Cũng có thể sửa đổi đễ thực hiện các hoạt động xử lý gói tin
```c
int  frame_processing(fsp_packet_t  *fsp_pkt){
switch  (fsp_pkt->type)
{
```
## Frame ví dụ:
Một Frame ví dụ có thể như sau:
```bash
{CA}{01}{05}{00}{05}{CD}{E1}{EF}
```
Trong đó
|  Giá trị| Định nghĩa |
|--|--|
| 0xCA | SOD - Start of Data|
| 0x01 | SRC - CPOC |
| 0x05 | DST - IOU |
| 0x00 | LEN = 0 |
| 0x05 | Type: Only ACK |
| 0xCD | CRC-16 MSB |
| 0xE1 | CRC-16 LSB |
| 0xEF | EOF - End of Frame |
