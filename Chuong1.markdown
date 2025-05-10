# Chương 1: Các Khái Niệm Cơ Bản

## Giới Thiệu

Khi nào bạn cần quan tâm đến quản lý bộ nhớ trong .NET nếu mọi thứ đều được tự động hóa? Có cần quan tâm hay không? Câu trả lời là có, và điều này rất quan trọng. Là một lập trình viên, tính chuyên nghiệp đòi hỏi chúng ta phải xem xét quản lý bộ nhớ trong mọi tình huống. Điều này không chỉ là hoàn thành công việc mà còn là làm thế nào để công việc được thực hiện tốt nhất—tối ưu hóa việc sử dụng CPU và bộ nhớ, đảm bảo tính dễ bảo trì, khả năng kiểm thử, và tuân thủ các nguyên tắc SOLID. Những mối quan tâm này phân biệt các lập trình viên mới bắt đầu với những người có kinh nghiệm, những người ưu tiên các khía cạnh không chức năng như hiệu suất và chất lượng.

Các vấn đề về bộ nhớ, chẳng hạn như `AccessViolationException` hoặc việc sử dụng bộ nhớ không kiểm soát, hiếm khi xảy ra trong .NET nhờ vào runtime mạnh mẽ của nó. Tuy nhiên, tiêu thụ bộ nhớ thường đứng đầu danh sách các vấn đề về hiệu suất trong các ứng dụng .NET lớn. Ví dụ, rò rỉ bộ nhớ trong một hệ thống chạy liên tục trong thời gian dài hoặc chi phí của quản lý bộ nhớ tự động trong các môi trường không máy chủ như Azure Functions (được tính phí theo gigabyte-giây) có thể gây ra hậu quả nghiêm trọng. Việc hiểu về quản lý bộ nhớ trong .NET là rất quan trọng để chẩn đoán và giải quyết các vấn đề như vậy.

Chương này cung cấp một nền tảng lý thuyết, thiết lập sự hiểu biết chung về các khái niệm liên quan đến bộ nhớ. Nó bao gồm bối cảnh lịch sử và tham chiếu đến các công nghệ cụ thể để làm cho cuộc thảo luận trở nên hấp dẫn và phù hợp. Mục tiêu là cung cấp một góc nhìn rộng về quản lý bộ nhớ, so sánh cách tiếp cận của .NET với các ngôn ngữ và runtime khác.

## Các Thuật Ngữ Liên Quan Đến Bộ Nhớ

Trước khi đi sâu hơn, hãy định nghĩa các thuật ngữ quan trọng cần thiết để thảo luận về bộ nhớ:

- **Bit**: Đơn vị thông tin nhỏ nhất, biểu thị hai trạng thái (0 hoặc 1, đúng hoặc sai). Các bit được kết hợp để tạo thành các đơn vị dữ liệu lớn hơn, với kích thước dữ liệu được ký hiệu bằng chữ cái thường 'b'.
- **Số Nhị Phân**: Giá trị số nguyên được biểu diễn dưới dạng một chuỗi các bit, trong đó mỗi bit đóng góp vào giá trị dưới dạng lũy thừa của 2. Ví dụ, số 5 được biểu diễn là 101 (1×1 + 0×2 + 1×4). Một số nhị phân n-bit có thể biểu diễn giá trị lên đến 2^n-1. Các bit bổ sung có thể mã hóa dấu hoặc số thực dấu chấm động.
- **Mã Nhị Phân**: Một chuỗi bit biểu thị dữ liệu như văn bản (ví dụ: ASCII, sử dụng mã 7-bit) hoặc opcode (lệnh cho máy tính).
- **Byte**: Một chuỗi bit, thường là 8 bit, được sử dụng để mã hóa một ký tự đơn. Mặc dù trong lịch sử phụ thuộc vào kiến trúc, byte 8-bit hiện là tiêu chuẩn, được ký hiệu bằng chữ cái in hoa 'B'. Một octet rõ ràng đề cập đến 8 bit.
- **Tiền Tố Kích Thước Dữ Liệu**: Các tiền tố phổ biến như kilo (1000, ký hiệu 'k'), mega (1,000,000, 'M'), và giga (1,000,000,000, 'G') được sử dụng cho kích thước dữ liệu. Trong máy tính, các bội số của 1024 thường được sử dụng (kibi 'Ki', mebi 'Mi', gibi 'Gi'), dẫn đến sự mơ hồ (ví dụ: 1 GB so với 1 GiB). Ngữ cảnh xác định cách giải thích đúng, vì ngay cả các tiêu chuẩn như JEDEC cũng cho phép sự linh hoạt.

## Bối Cảnh Lịch Sử

Các máy tính hiện đại có RAM đáng kể (ví dụ: 8 GiB trong đồng hồ thông minh), nhưng các máy tính ban đầu chỉ có bộ nhớ tối thiểu. **Z3** (1941, Konrad Zuse) là một máy tính cơ điện với 176 byte bộ nhớ (64 ô 22-bit). **Harvard Mark I** (1944) cung cấp 702 byte qua 72 bộ tích lũy 78-bit. Những máy này thiếu quản lý bộ nhớ tinh vi, chỉ cho phép các thao tác tải/lưu cơ bản.

**ENIAC** (1946) sử dụng ống chân không để tính toán nhanh hơn nhưng có bộ nhớ hạn chế (20 bộ tích lũy 10 chữ số). **Manchester SSEM ("Baby")** (1948) và **EDSAC** (1949) giới thiệu **kiến trúc von Neumann**, lưu trữ chương trình và dữ liệu trong cùng một bộ nhớ (128 byte RAM trong SSEM sử dụng ống Williams). Những tiến bộ này đã đặt nền móng cho máy tính hiện đại.

### Các Khái Niệm Kiến Trúc Máy Tính

- **Bộ Nhớ**: Lưu trữ dữ liệu và chương trình, phát triển từ thẻ đục lỗ đến transistor. Bao gồm:
  - **Bộ Nhớ Truy Cập Ngẫu Nhiên (RAM)**: Cung cấp thời gian truy cập đồng đều (xấp xỉ, do bộ đệm hiện đại).
  - **Bộ Nhớ Truy Cập Không Đồng Đều**: Thời gian truy cập phụ thuộc vào vị trí vật lý (ví dụ: ổ cứng).
- **Địa Chỉ**: Một vị trí bộ nhớ cụ thể, thường có độ chi tiết byte.
- **Đơn Vị Số Học và Logic (ALU)**: Thực hiện các phép tính như cộng.
- **Đơn Vị Điều Khiển**: Giải mã các lệnh (opcode) và chỉ đạo các hoạt động.
- **Thanh Ghi**: Bộ nhớ nhanh trong các đơn vị thực thi (ALU/đơn vị điều khiển), ví dụ: bộ tích lũy.
- **Word**: Đơn vị dữ liệu kích thước cố định (ví dụ: 32 hoặc 64 bit trong máy tính hiện đại), xác định kích thước thanh ghi và bộ nhớ có thể định địa chỉ.
- **Con Trỏ Lệnh (IP)/Bộ Đếm Chương Trình (PC)**: Theo dõi địa chỉ của lệnh hiện tại.

**Máy tính lưu trữ chương trình** (kiến trúc von Neumann) lưu trữ lệnh và dữ liệu cùng nhau, cho phép lập trình linh hoạt. Lập trình ban đầu sử dụng mã nhị phân, phát triển thành ngôn ngữ assembly (2GL) và các ngôn ngữ cấp cao (3GL) như C, được biên dịch thành opcode nhị phân.

## Các Chiến Lược Phân Bổ Bộ Nhớ

### Phân Bổ Tĩnh

Các ngôn ngữ ban đầu (ví dụ: FORTRAN, ALGOL) sử dụng phân bổ tĩnh, nơi kích thước và vị trí bộ nhớ được cố định tại thời điểm biên dịch. Điều này đơn giản nhưng không linh hoạt, yêu cầu biên dịch lại cho các kích thước dữ liệu khác nhau. Phân bổ tĩnh vẫn tồn tại trong các hệ thống hiện đại cho các biến toàn cục.

### Máy Thanh Ghi

Máy thanh ghi (ví dụ: CPU x86) thực hiện các phép tính trên thanh ghi. Ví dụ, tính toán `s = x + (2*y) + z` bao gồm việc tải dữ liệu vào thanh ghi, thực hiện các phép toán, và lưu kết quả. Điều này trái ngược với các thao tác trực tiếp trên bộ nhớ (ví dụ: IBM System/360), hiện nay rất hiếm.

```plaintext
Load A, y     // A = y
Multiply A, 2 // A = 2*y
Load B, x     // B = x
Add A, B      // A = x + 2*y
Load B, z     // B = z
Add A, B      // A = x + 2*y + z
Store s, A    // s = A
```

### Ngăn Xếp (Stack)

Ngăn xếp là một cấu trúc dữ liệu Vào Sau Ra Trước (LIFO), hỗ trợ các thao tác đẩy (push - thêm) và lấy (pop - xóa). Nó trở nên quan trọng với các chương trình con, cho phép gọi hàm lồng nhau. **EDSAC** (David Wheeler) giới thiệu "Wheeler jump" cho địa chỉ trả về, phát triển thành khái niệm ngăn xếp bởi Alan Turing (ACE, 1940s) với các lệnh BURY/UNBURY.

Trong .NET, ngăn xếp lưu trữ các khung kích hoạt (stack frame) cho các lần gọi hàm, bao gồm đối số, địa chỉ trả về, và biến cục bộ. Ví dụ:

```c
void main() {
    fun1(2, 3);
}
int fun1(int a, int b) {
    int x, y;
    fun2(a + b);
}
int fun2(int n) {
    int z;
}
```

Ngăn xếp phát triển xuống dưới, với mỗi lần gọi hàm thêm một khung kích hoạt. Gọi lồng quá mức có thể gây ra `StackOverflowException`. Khi hàm trả về, khung của nó được loại bỏ bằng cách điều chỉnh con trỏ ngăn xếp.

### Máy Ngăn Xếp

Máy ngăn xếp hoạt động trên một ngăn xếp biểu thức, lấy các toán hạng và đẩy kết quả. Ví dụ, tính toán `s = x + (2*y) + z`:

```plaintext
Push 2        // [2]
Push y        // [2][y]
Multiply      // [2*y]
Push x        // [2*y][x]
Add           // [2*y+x]
Push z        // [2*y+x][z]
Add           // [2*y+x+z]
Pop s         // []
```

Máy ngăn xếp (ví dụ: .NET CLR, JVM) lý tưởng cho các máy ảo do tính đơn giản và độc lập với nền tảng. Chúng trừu tượng hóa các thanh ghi phần cứng, tạo ra mã dày đặc, thân thiện với bộ đệm. Ngôn ngữ trung gian của .NET (IL) hoạt động như một máy ngăn xếp.

### Con Trỏ (Pointer)

Con trỏ, được giới thiệu trong PL/I (1965), lưu trữ địa chỉ bộ nhớ, cho phép các cấu trúc dữ liệu động như danh sách liên kết. Chúng thường là 32 hoặc 64 bit, được lưu trữ trên ngăn xếp hoặc thanh ghi. Phép toán con trỏ cho phép truy cập bộ nhớ tương đối. Trong .NET, con trỏ bị hạn chế để đảm bảo an toàn, vì quản lý thủ công có thể dẫn đến lỗi.

### Đống (Heap)

Đống (hoặc kho lưu trữ tự do) hỗ trợ phân bổ bộ nhớ động, nơi kích thước và vị trí được xác định tại thời điểm chạy. Được giới thiệu trong ALGOL 68, đống là một "kho" bộ nhớ lộn xộn được truy cập qua con trỏ. Phân bổ và giải phóng là rõ ràng trong quản lý thủ công, nhưng .NET tự động hóa điều này.

Phân mảnh đống xảy ra khi bộ nhớ trống không liền mạch, làm giảm hiệu quả. Ngăn xếp và đống khác nhau đáng kể:

| **Thuộc Tính**       | **Ngăn Xếp**                                  | **Đống**                                     |
|----------------------|-----------------------------------------------|---------------------------------------------|
| **Vòng Đời**         | Dựa trên phạm vi (đẩy vào/lấy ra)              | Rõ ràng (phân bổ/giải phóng)                |
| **Phạm Vi**          | Cục bộ (luồng)                                | Toàn cục (qua con trỏ)                      |
| **Truy Cập**         | Biến cục bộ, đối số                           | Con trỏ                                     |
| **Thời Gian Truy Cập**| Nhanh (bộ đệm)                               | Chậm hơn (có thể được lưu trữ tạm trên đĩa)  |
| **Phân Bổ**         | Di chuyển con trỏ ngăn xếp                    | Các chiến lược khác nhau                    |
| **Thời Gian Phân Bổ**| Rất nhanh                                    | Chậm hơn                                    |
| **Giải Phóng**       | Di chuyển con trỏ ngăn xếp                    | Các chiến lược khác nhau                    |
| **Sử Dụng**          | Tham số, biến cục bộ, mảng nhỏ                | Mọi thứ                                     |
| **Dung Lượng**       | Hạn chế (vài MB mỗi luồng)                    | Gần như không giới hạn                      |
| **Kích Thước Biến**  | Không                                         | Có                                          |
| **Phân Mảnh**        | Không                                         | Có thể xảy ra                               |
| **Mối Đe Dọa Chính** | Tràn ngăn xếp                                | Rò rỉ bộ nhớ, phân mảnh                     |

## Quản Lý Bộ Nhớ Thủ Công vs. Tự Động

Quản lý bộ nhớ thủ công yêu cầu lập trình viên phân bổ và giải phóng bộ nhớ một cách rõ ràng, tương tự như chuyển số tay trong ô tô. Điều này mang lại sự kiểm soát nhưng dễ xảy ra lỗi. Quản lý bộ nhớ tự động, như trong .NET, trừu tượng hóa bộ nhớ như vô hạn, với runtime xử lý giải phóng.

### Các Thành Phần Chính

- **Mutator**: Thực thi mã ứng dụng, thực hiện các thao tác như `New` (phân bổ), `Write` (gán), và `Read` (truy cập). Trong .NET, mutator thường là các luồng.
- **Allocator**: Quản lý phân bổ và giải phóng bộ nhớ động (`Allocate` và `Deallocate`). .NET sử dụng các bộ phân bổ tuần tự hoặc danh sách tự do.
- **Collector (Bộ Thu Gom Rác)**: Thu hồi bộ nhớ từ các đối tượng không thể truy cập. Nó xấp xỉ tính sống của đối tượng qua khả năng truy cập từ các gốc (ví dụ: biến ngăn xếp, đối tượng toàn cục).

### Đếm Tham Chiếu (Reference Counting)

Đếm tham chiếu theo dõi số lượng tham chiếu đến một đối tượng, giải phóng nó khi số đếm đạt 0. Ví dụ:

```c
Mutator.New(amount) {
    obj = Allocator.Allocate(amount);
    obj.counter = 0;
    return obj;
}
Mutator.Write(address, value) {
    if (address != NULL)
        ReferenceCountingCollector.DecreaseCounter(address);
    *address = value;
    if (value != NULL)
        value.counter++;
}
ReferenceCountingCollector.DecreaseCounter(address) {
    *address.counter--;
    if (*address.counter == 0)
        Allocator.Deallocate(address);
}
```

**Ưu điểm**:
- Giải phóng xác định.
- Chi phí bộ nhớ thấp.
- Độc lập với runtime (ví dụ: con trỏ thông minh trong C++).

**Nhược điểm**:
- Chi phí trên các phép gán.
- Chi phí đồng bộ hóa đa luồng.
- Tham chiếu vòng gây rò rỉ.

C++ sử dụng `unique_ptr` (sở hữu duy nhất) và `shared_ptr` (đếm tham chiếu) để quản lý tự động:

```cpp
#include <iostream>
#include <memory>
void printReport(std::shared_ptr<int> data) {
    std::cout << "Báo cáo: " << *data << "\n";
}
int main() {
    try {
        std::shared_ptr<int> ptr(new int());
        *ptr = 25;
        printReport(ptr);
        return 0;
    } catch (std::bad_alloc& ba) {
        std::cout << "LỖI: Hết bộ nhớ\n";
        return 1;
    }
}
```

### Bộ Thu Gom Rác Theo Dõi (Tracking Garbage Collector)

Bộ thu gom rác theo dõi xác định khả năng truy cập bằng cách duyệt qua đồ thị đối tượng từ các gốc, đánh dấu các đối tượng có thể truy cập và thu hồi các đối tượng khác. Bao gồm:

- **Giai Đoạn Đánh Dấu (Mark Phase)**: Đánh dấu các đối tượng có thể truy cập, xử lý các tham chiếu vòng.
- **Giai Đoạn Thu Gom (Collect Phase)**:
  - **Quét (Sweep)**: Đánh dấu các đối tượng chết là không gian trống (nhanh nhưng gây phân mảnh).
  - **Nén (Compact)**: Di chuyển các đối tượng để loại bỏ khoảng trống (chậm hơn nhưng giảm phân mảnh).
    - **Sao Chép (Copying)**: Sao chép các đối tượng sống sang vùng mới.
    - **Tại Chỗ (In-Place)**: Di chuyển các đối tượng trong cùng vùng.

**Ưu điểm**:
- Minh bạch với lập trình viên.
- Xử lý tham chiếu vòng.
- Chi phí mutator thấp.

**Nhược điểm**:
- Triển khai phức tạp.
- Giải phóng không xác định.
- Có thể tạm dừng (phi đồng thời).
- Sử dụng bộ nhớ cao hơn.

### Bộ Thu Gom Rác Bảo Thủ vs. Chính Xác

- **Bộ Thu Gom Rác Bảo Thủ**: Đoán con trỏ bằng cách quét bộ nhớ, giả định các địa chỉ hợp lệ là tham chiếu. Được sử dụng trong các môi trường thiếu thông tin kiểu (ví dụ: Boehm GC cho C++). Nó không chính xác và ngăn chặn di chuyển đối tượng.
- **Bộ Thu Gom Rác Chính Xác**: Sử dụng thông tin kiểu do runtime cung cấp để theo dõi gốc và tham chiếu chính xác. .NET sử dụng bộ thu gom rác chính xác.

### So Sánh Các Bộ Thu Gom Rác

So sánh các bộ thu gom rác (ví dụ: .NET so với JVM) là thách thức do tích hợp runtime. Các chỉ số như thông lượng, độ trễ và thời gian tạm dừng phụ thuộc vào khối lượng công việc, phần cứng và tối ưu hóa. .NET sử dụng bộ thu gom rác chính xác, đánh dấu-lập kế hoạch-quét-nén, được tối ưu hóa cho nhiều kịch bản.

## Góc Nhìn Lịch Sử

- **LISP**: Một trong những ngôn ngữ sớm nhất có bộ thu gom rác, hiện được sử dụng trong các phương ngữ như Clojure.
- **Simula**: Giới thiệu OOP và bộ thu gom rác nén, ảnh hưởng đến C++, Java và .NET.
- **Java**: Phổ biến bộ thu gom rác với đánh dấu và quét vào những năm 1990.
- **Python/Ruby**: Áp dụng bộ thu gom rác cho lập trình cấp cao.
- **JavaScript**: Sử dụng đánh dấu-quét-nén trong V8 cho các ứng dụng web.
- **.NET**: Phát triển từ bộ thu gom rác bảo thủ của JScript sang bộ thu gom rác chính xác, lấy cảm hứng từ LISP, được thiết kế cho lập trình hướng thành phần bởi Patrick Dussud.

## Quy Tắc 1: Tự Học Hỏi

**Áp dụng**: Phổ quát.

**Lý Do**: Học hỏi liên tục là điều cần thiết cho tính chuyên nghiệp. Hiểu biết về các hệ thống cơ bản (ví dụ: quản lý bộ nhớ của .NET) nâng cao hiệu suất và khả năng giải quyết vấn đề, tương tự như “Đồng Cảm Cơ```plaintext
Cảm Hứng Cơ Học” trong đua xe—cảm nhận cách hệ thống hoạt động.

**Cách Áp Dụng**:
- Đọc sách và bài báo.
- Tham dự hội nghị hoặc các nhóm người dùng địa phương.
- Tham gia các khóa học trực tuyến.
- Viết blog để dạy người khác.
- Khám phá các nguyên tắc của Software Craftsmanship.

Chương này đặt nền tảng cho việc hiểu về quản lý bộ nhớ trong .NET, kết nối lý thuyết và triển khai thực tế trong các chương tiếp theo.