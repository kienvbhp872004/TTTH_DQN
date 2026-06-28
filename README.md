# TTTH_DQN

Project mô phỏng bài toán điều hướng robot trên lưới 2D bằng các controller học tăng cường và các baseline cổ điển. Ứng dụng chạy bằng `pygame`, hỗ trợ `train` và `test`, lưu model, biểu đồ và log đường đi theo từng map / controller.

## Tổng Quan

- Môi trường kích thước `512 x 512`, chia thành lưới `32 x 32` với ô `16 px`.
- Robot bắt đầu từ vị trí `Start`, di chuyển tới `Goal`, đồng thời tránh chướng ngại vật tĩnh và động.
- Project có nhiều controller khác nhau để so sánh hiệu năng.
- Trong chế độ training, project lưu model tốt nhất, metrics theo từng episode và biểu đồ reward / episode length.
- Trong chế độ test, project lưu đường đi của robot để phục vụ phân tích sau đó.

## Tính Năng Chính

- Giao diện mô phỏng trực quan bằng `pygame`.
- Chọn `train` hoặc `test` ngay khi chạy chương trình.
- Hỗ trợ nhiều controller:
  - `DQNController`
  - `DFQLController`
  - `DWAController`
  - `DQNPMController`
  - `SDQNController`
  - `PER_DQNController`
  - `TransformerDQNController`
- Hỗ trợ thêm các baseline cổ điển qua `main_classical.py`:
  - `SmartAvoidanceController`
  - `DeterministicRRTController`
- Lưu model theo từng map / controller.
- Tự động lưu `best_model_*.pth` khi training cải thiện.
- Ghi lại path trong `results/` để phân tích chất lượng đường đi.

## Cấu Trúc Project

```text
.
|-- main.py                  # Entry point chính: train/test với nhiều controller
|-- main_classical.py        # Chạy các controller cổ điển
|-- MapData.py               # Danh sách map, Start, Goal, Obstacles
|-- Robot.py                 # Logic robot và trạng thái quan sát
|-- Obstacle.py              # Chướng ngại vật tĩnh / động
|-- Colors.py                # Hằng số màu sắc cho giao diện
|-- controller/              # Các controller DQN, PER-DQN, Transformer, DWA, ...
|-- utils/                   # Tiện ích phụ trợ, ví dụ chọn GPU
|-- models/                  # Model đã train
|-- plots/                   # Biểu đồ training
|-- results/                 # Log đường đi trong test / training
|-- metrics/                 # Kết quả so sánh metrics
|-- result_visualized/       # Ảnh tổng hợp đường đi / thống kê
|-- map_images/              # Ảnh minh hoạ map
|-- README.md                # Tài liệu này
```

## Map Có Sẵn

Các map hiện có trong `MapData.py`:

- `uniform1`
- `uniform2`
- `uniform3`
- `diverse1`
- `diverse2`
- `diverse3`
- `complex1`
- `complex2`
- `complex3`

## Yêu Cầu

- Python 3.10+ khuyến nghị
- `pygame`
- `numpy`
- `matplotlib`
- `torch`
- Các thư viện khác được liệt kê trong `requirements.txt`

> Nếu dùng GPU, hãy cài bản `torch` phù hợp với CUDA của máy bạn. Nếu chỉ chạy CPU, có thể cài bản CPU tương ứng.

## Cài Đặt

### 1. Clone project

```bash
git clone https://github.com/jien0404/TTTH_DQN.git
cd TTTH_DQN
```

### 2. Tạo môi trường ảo

```bash
python -m venv .venv
```

Kích hoạt:

```bash
# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

### 3. Cài dependencies

Nếu bạn chưa có `torch`, cài trước phiên bản phù hợp với máy:

```bash
pip install torch torchvision torchaudio
```

Sau đó cài các thư viện còn lại:

```bash
pip install -r requirements.txt
```

## Cách Chạy

### Chạy mô phỏng chính

```bash
python main.py
```

Luồng thao tác:

1. Chọn `train` hoặc `test`.
2. Chọn controller.
3. Chọn map.
4. Nếu `train`, nhập số episode huấn luyện.
5. Nếu `test`, chọn model đã lưu và số episode kiểm tra.

### Chạy baseline cổ điển

```bash
python main_classical.py
```

File này dùng cho các controller cổ điển trong `controller/` và luồng test rút gọn.

## Output Sinh Ra

Sau khi chạy, project sẽ tạo các thư mục theo cấu trúc:

- `models/<map>/<controller>/`
  - file model `.pth`
  - `best_model_*.pth`
- `plots/<map>/<controller>/`
  - biểu đồ reward
  - biểu đồ độ dài episode
- `results/<map>/<controller>/`
  - log đường đi robot theo episode
- `metrics/<map>/`
  - bảng và biểu đồ so sánh metrics
- `result_visualized/<map>/`
  - ảnh tổng hợp đường đi / thống kê

## Ghi Chú Khi Dùng

- `DWAController` chỉ chạy ở chế độ `test`.
- Khi training, model được cập nhật theo từng run và có thể được lưu định kỳ.
- Nếu không thấy model trong chế độ `test`, hãy train trước trên đúng `map` và `controller` đó.
- Một số script phụ trợ trong repo:
  - `compare_metrics.py`
  - `visualize_best_paths.py`
  - `MapCreator.py`
  - `MapView.py`
  - `metrics.py`
  - `auto_test.py`
  - `Auto_Run.py`

## Gợi Ý Mở Rộng

- Thử so sánh cùng một map giữa nhiều controller để tìm cấu hình tốt nhất.
- Dùng `PER_DQNController` hoặc `TransformerDQNController` nếu muốn thử biến thể mạnh hơn DQN cơ bản.
- Phân tích thêm file trong `results/` để đo độ dài đường đi, số lần va chạm và tỷ lệ tới đích.

## Tác Giả

Project này là một môi trường nghiên cứu / thực nghiệm cho điều hướng robot bằng học tăng cường. Nếu bạn muốn, mình có thể giúp viết tiếp phần `Usage`, `Troubleshooting` hoặc thêm badge / ảnh minh hoạ cho README.
