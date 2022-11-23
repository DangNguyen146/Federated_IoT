# fed_iot_guard
Phát hiện các thiết bị IoT bị nhiễm phần mềm độc hại từ giao tiếp mạng của chúng, sử dụng máy học liên kết

Mã này cho phép chạy các thử nghiệm mô phỏng các cấu hình khác nhau của client_iot đang cố gắng đào tạo các mô hình học sâu để phát hiện phần mềm độc hại trong thiết bị IoT của họ. Một phần lớn của mã được dành riêng để chạy các thử nghiệm federated learning để client_iot có thể đào tạo mô hình của họ một cách cộng tác mà không phải chia sẻ bất kỳ dữ liệu nào. Vì đây là mô phỏng nên mọi thứ chạy cục bộ trên máy của bạn, sử dụng tập dữ liệu công khai N-BaIoT (https://archive.ics.uci.edu/ml/datasets/detection_of_IoT_botnet_attacks_N_BaIoT). Ấn phẩm tương ứng với mã này hiện có tại https://www.sciencedirect.com/science/article/pii/S1389128621005582 ở chế độ truy cập mở.

## Thành lập
### Bước 0: khuyến nghị:
* Sử dụng Linux (mình dùng Ubuntu 20.04) để dễ dàng download và giải nén dataset hơn
* Sử dụng Python 3.9 để tương thích
* Sử dụng PyCharm (có một phiên bản miễn phí có tên là Cộng đồng PyCharm, phiên bản này đủ tốt), để bạn có ngay cấu hình chạy (chứa các đối số chương trình) mà tôi đã sử dụng cho các thử nghiệm của mình. Mặt khác, bạn có thể hiểu cách thức hoạt động của các đối số chương trình và thậm chí bạn có thể chạy trực tiếp chương trình từ device endpoint.

### Bước 1: lấy kho lưu trữ
Sao chép hoặc tải kho lưu trữ về máy của bạn.

### Bước 2: lấy dữ liệu
Tệp .gitignore chứa thư mục `data/`, vì vậy bạn phải tạo thư mục này theo cách thủ công, tải xuống và giải nén tập dữ liệu vào đó.
* Tạo thư mục `data/` trong `fed_iot_guard/`
* Tạo thư mục `N-BaIoT/` bên trong `data/`
* Từ thư mục `N-BaIoT/`, nếu bạn đang dùng linux, hãy mở terminal và chạy lệnh sau: `wget -r -np -nH --cut-dirs=3 -R "index.html*" https://archive.ics.uci.edu/ml/machine-learning-databases/00442/`

  Nó sẽ tải xuống tập dữ liệu. Lệnh này sẽ mất vài phút để thực hiện, tùy thuộc vào kết nối internet của bạn.
  Nếu bạn đang sử dụng hệ điều hành khác với Linux hoặc nếu bạn không thể sử dụng wget vì lý do khác, bạn cần tìm một công cụ thay thế để tải xuống tệp đệ quy từ https://archive.ics.uci.edu/ml/machine-learning -databases/00442/ hoặc bạn cần tải xuống tất cả chúng theo cách thủ công.

* Bây giờ dữ liệu đã được tải xuống, nhưng một số tệp bên trong các thư mục bên trong vẫn ở định dạng lưu trữ (đuôi .rar). Để khắc phục điều đó, tôi đã sử dụng lệnh `unar`, lệnh này có thể dễ dàng cài đặt trên linux bằng cách sử dụng `sudo apt install unar`. Để chạy lệnh một cách đệ quy, hãy chạy `find ./ -name '*.rar' -execdir unar {} \;` từ thư mục `N-BaIoT/`.
  Một lần nữa, nếu bạn không thể sử dụng unar vì lý do nào đó, bạn cũng có thể hủy giải mã thủ công từng tệp .rar trong bộ dữ liệu.

### Bước 3: cài đặt các thư viện python cần thiết.

* Sử dụng PyCharm: tạo một trình thông dịch python mới cho dự án ("Tệp > Cài đặt > Dự án: fed_iot_guard > Trình thông dịch Python > Biểu tượng bánh răng > Thêm > Môi trường Virtualenv hoặc bất cứ thứ gì bạn thích > Chọn một vị trí mà bạn sẽ tạo môi trường mới của mình và chọn Python 3.9 làm trình thông dịch cơ sở của bạn). Khi điều này được thực hiện, PyCharm sẽ thấy rằng các yêu cầu được liệt kê trong tests.txt không thỏa mãn và nó sẽ yêu cầu cài đặt các yêu cầu đã nói (bạn nên làm điều đó). Khi bạn đã thực hiện xong, bạn có thể chạy bất kỳ cấu hình nào từ PyCharm (ví dụ: `fedavg_autoencoders_test`).

* Sử dụng virtualenv: Tạo một môi trường ảo mới dựa trên Python 3.9. Kích hoạt môi trường này. Cài đặt các yêu cầu bằng cách di chuyển đến thư mục `fed_iot_guard/` và chạy `pip install -r tests.txt`. Sau đó, bạn có thể chạy chương trình chính từ device endpoint. Chẳng hạn, bạn có thể chạy (từ thư mục `fed_iot_guard/`): `python src/main.py bộ mã hóa tự động phi tập trung --test --fedavg --collaborative --verbose-depth=1`.

## Cách sử dụng

### Đối số chương trình
Để chạy thử nghiệm, bạn phải chạy main.py với các đối số thích hợp. Vì có rất nhiều cấu hình khác nhau để thử nghiệm và các cách tiếp cận khác nhau đối với tất cả chúng, nên có khá nhiều tham số cho phép chúng tôi thực hiện điều đó một cách nghiêm ngặt.
* Tham số đầu tiên xác định xem bạn coi dữ liệu của client_iot là phi tập trung (mỗi client_iot giữ dữ liệu của riêng mình) hay tập trung (dữ liệu được đặt chung). Các giá trị được chấp nhận là `phi tập trung` và `tập trung`.
* Tham số thứ hai xác định xem bạn muốn thực hiện học có giám sát (giả sử mỗi client_iot có quyền truy cập vào một số dữ liệu được gắn nhãn của cả lớp lành tính và độc hại) bằng cách sử dụng bộ phân loại mạng thần kinh hay bạn muốn học không giám sát (giả sử mỗi client_iot chỉ có quyền truy cập vào một số dữ liệu lành tính) bằng cách sử dụng bộ mã hóa tự động. Do đó, các giá trị được chấp nhận cho tham số này là `bộ phân loại` và `bộ mã hóa tự động`.
* Là tham số thứ ba, bạn chỉ định `--gs` nếu bạn muốn chạy tìm kiếm dạng lưới để chọn tập hợp siêu tham số tốt nhất (sử dụng dữ liệu huấn luyện để huấn luyện mô hình và dữ liệu xác thực để đánh giá nó) hoặc `- -test` nếu bạn muốn đào tạo mô hình bằng cách sử dụng dữ liệu đào tạo và xác thực và đánh giá nó trên tập kiểm tra (để obtain là kết quả kiểm tra cuối cùng sau khi bạn đã chọn sử dụng siêu tham số nào).
* Trong trường hợp bạn đã chọn `--decentralized`, bạn có thể cho phép client_iot cộng tác bằng cách sử dụng `--collaborative` hoặc không, bằng cách sử dụng `--no-collaborative`. Khi chạy tìm kiếm dạng lưới (`--gs`), tính năng cộng tác cho phép client_iot chọn siêu tham số mang lại kết quả trung bình tốt nhất trong số chúng (chứ không phải tất cả client_iot đều có tập hợp siêu tham số tốt nhất của riêng mình. Khi chạy tìm kiếm thử nghiệm cuối cùng (`--test`), sự cộng tác cho phép sử dụng phương pháp học liên kết thay vì yêu cầu mỗi client_iot đào tạo mô hình của riêng mình một cách riêng biệt.
* Trong trường hợp bạn đã chọn `--test` và `--collaborative`, bạn vẫn có thể quyết định loại thuật toán liên kết mà bạn muốn sử dụng. Nội dung được gọi là `Tập hợp lô nhỏ` trong ấn phẩm được gọi là `fedsgd` trong mã và nội dung được gọi là `Tập hợp nhiều giai đoạn` trong ấn phẩm được gọi là `fedavg` trong mã. Lý do là cách đặt tên được sử dụng trong mã gây hiểu nhầm (các phiên bản fedsgd và fedavg của tôi không hoàn toàn tương đương với những gì được định nghĩa trong "Học hiệu quả giao tiếp của các mạng sâu từ dữ liệu phi tập trung") nên tôi đã quyết định thay đổi tên của chúng trong ấn phẩm, nhưng tôi đã không cập nhật mã cho phù hợp. Xin lỗi vì chuyện đó. Tóm lại, hãy sử dụng `--fedsgd` cho `Tập hợp lô nhỏ` hoặc `--fedavg` cho `Tập hợp nhiều kỷ nguyên`.
* Hai tham số cuối cùng là về các bản in trong bảng điều khiển. `--verbose` kích hoạt in trong bảng điều khiển về trạng thái của quá trình đào tạo (được khuyến nghị vì các thử nghiệm có thể chạy rất lâu). `--no-verbose` hủy kích hoạt tất cả các bản in. Ngoài ra, bạn có thể xác định độ sâu tối đa của các vòng lặp bên trong mà tính năng in được kích hoạt bằng cách sử dụng `--verbose-depth n` với `n` là một số nguyên. Nên sử dụng giá trị 1, 2 hoặc 3.

Ví dụ (các lệnh chạy từ device endpoint trong `fed_iot_guard/`, với môi trường python thích hợp được kích hoạt):
* Tìm kiếm lưới cộng tác giữa các client_iot phi tập trung cho các trình phân loại: `python src/main.py trình phân loại phi tập trung --gs --collaborative --verbose-depth=2`
* Đào tạo và thử nghiệm liên kết giữa các client_iot phi tập trung cho bộ mã hóa tự động, sử dụng `Tập hợp nhiều kỷ nguyên`: `python src/main.py bộ mã hóa tự động phi tập trung --test --fedavg --collaborative --verbose-depth=2`

Lưu ý rằng nếu bạn đang sử dụng PyCharm, bạn sẽ có quyền truy cập trực tiếp vào tất cả các cấu hình mà tôi đã sử dụng cho các thử nghiệm của mình. Các cấu hình này được lưu trong thư mục `.idea/runConfigurations/`, ở dạng tệp .xml. Nếu bạn không sử dụng PyCharm, bạn có thể kiểm tra các tệp này và xem giá trị `PARAMETERS` để nhận tất cả các tham số của cấu hình mà tôi đã sử dụng.

### Siêu tham số
Nhiều siêu tham số khác nhau được xác định trong mã của hàm chính, bên trong từ điển. Một số trong số chúng là cố định (bên trong `constant_params`) và một số thay đổi trong quá trình tìm kiếm lưới (bên trong `varying_params`). Khi bạn đã chạy tìm kiếm dạng lưới và thu được kết quả, bạn có thể sử dụng sổ ghi chép jupyter `GS Results.ipynb` để kiểm tra siêu tham số nào là tốt nhất cho từng cấu hình. Sau đó, bạn có thể sao chép các siêu tham số thu được cho từng cấu hình bên trong danh sách `configuration_params` thích hợp (có một danh sách như vậy dành cho bộ mã hóa tự động và một danh sách dành cho bộ phân loại).

## Sửa đổi
Để làm cho mã của tôi có thể mở rộng dễ dàng, sau đây là một số giải thích về hoạt động bên trong của nó, theo từng tệp.
* `main.py` xử lý các đối số chương trình, xác định siêu tham số, tạo cấu hình thích hợp của client_iot và gọi các hàm thử nghiệm thích hợp.
* `data.py` làm mọi thứ liên quan đến dữ liệu: đọc, lấy mẫu lại và chia tách chẳng hạn.
* `supervised_data.py` chứa các thao tác cần thiết để lấy bộ dữ liệu được giám sát.
* `unsupervised_data.py` chứa các thao tác cần thiết để lấy bộ dữ liệu không giám sát.
* `architectures.py` xác định kiến ​​trúc PyTorch được sử dụng (bộ phân loại mạng thần kinh và bộ mã hóa tự động).
* `ml.py` xác định một số chức năng máy học phổ biến giữa các phương pháp được giám sát và không được giám sát (công cụ chuẩn hóa).
* `supervised_ml.py` chứa các hàm để huấn luyện bộ phân loại PyTorch.
* `unsupervised_ml.py` chứa các hàm để huấn luyện bộ mã hóa tự động PyTorch.
* `supervised_experiments.py` chứa tất cả thử nghiệm thiết lập được giám sát, cục bộ hoặc liên kết. Lưu ý rằng đào tạo địa phương xử lý cả trường hợp tập trung và phi tập trung (không có sự cộng tác).
* `unsupervised_experiments.py` chứa tất cả các thử nghiệm thiết lập không giám sát, cục bộ hoặc liên kết. Lưu ý rằng đào tạo địa phương xử lý cả trường hợp tập trung và phi tập trung (không có sự cộng tác).
* `federated_util.py` chứa nhiều chức năng hữu ích cho việc học liên kết: chức năng tổng hợp, tấn công đối thủ.
* `grid_search.py` chứa tất cả mã dành riêng cho tìm kiếm dạng lưới.
* `test_hparams.py` chứa mã để thực hiện quá trình đào tạo và đánh giá cuối cùng của mô hình 