# TTTH_DQN

TTTH_DQN là một project mô phỏng bài toán điều hướng robot trên lưới 2D, trong đó robot phải đi từ điểm `Start` đến `Goal` và tránh chướng ngại vật tĩnh lẫn động. Project này tập trung vào so sánh nhiều controller học tăng cường khác nhau trong cùng một môi trường, đồng thời lưu lại model, đường đi và biểu đồ để phân tích kết quả sau khi chạy.

## Bài Toán

Môi trường của bài toán là một bản đồ lưới với kích thước `512 x 512` pixel, chia thành lưới `32 x 32`, mỗi ô có kích thước `16 px`.

Robot được đặt ở vị trí xuất phát cố định, sau đó phải tìm đường đến đích trong khi:

- tránh va chạm với vật cản
- hạn chế đi vòng lặp hoặc bị kẹt tại một vùng
- ưu tiên đường đi ngắn, ổn định và ít góc ngoặt

Đây là một bài toán điều hướng robot điển hình trong học tăng cường, nơi agent phải cân bằng giữa:

- khám phá không gian trạng thái
- khai thác kinh nghiệm đã học
- tối ưu quãng đường và độ mượt của đường đi
- tránh các trạng thái thất bại như collision hoặc trapped

## Mô Hình Điều Hướng

### Quan Sát

Robot không nhìn toàn bộ bản đồ theo kiểu “global planner”, mà dùng một quan sát cục bộ để quyết định bước đi tiếp theo.

Cụ thể, trạng thái đầu vào của controller gồm:

- một vùng lân cận `5 x 5` quanh robot
- khoảng cách từ robot đến goal

Tổng kích thước state là `26 chiều`:

- `25` phần tử từ ma trận `5 x 5`
- `1` phần tử khoảng cách đến goal

### Hành Động

Không gian hành động là `8 hướng` rời rạc:

- Đông
- Đông Bắc
- Bắc
- Tây Bắc
- Tây
- Tây Nam
- Nam
- Đông Nam

### Hàm Phần Thưởng

Phần thưởng được thiết kế để khuyến khích robot đi đúng hướng và tránh hành vi kém hiệu quả:

- thưởng lớn khi chạm goal
- phạt mạnh khi va chạm
- thưởng khi tiến gần goal
- phạt nhẹ theo từng bước để ưu tiên đường ngắn
- phạt lặp vị trí để giảm vòng lặp
- cộng thêm curiosity reward để khuyến khích khám phá các vùng ít ghé thăm

### Chiến Lược Học

Các controller DQN trong repo dùng các kỹ thuật chính sau:

- `Dueling DQN`
- `Double DQN`
- `Experience Replay`
- `Target Network`
- `epsilon-greedy`
- cơ chế phạt lặp vị trí / tránh trap

## Các Controller Đã Có

### Nhóm DQN

- `DQNController`
- `PER_DQNController`
- `SDQNController`
- `DQNPMController`
- `TransformerDQNController`
- `DFQLController`
- `DWAController`

### Nhóm Baseline / Cổ Điển

- `SmartAvoidanceController`
- `DeterministicRRTController`

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

Mỗi map gồm:

- `Start`
- `Goal`
- danh sách `Obstacles`

Trong đó có cả chướng ngại vật tĩnh và chướng ngại vật động.

## Kết Quả Đã Chạy Được

Phần dưới đây được tổng hợp từ các file kết quả có sẵn trong repo:

- `metrics/uniform1/stats_20251219_230529.csv`
- `metrics/diverse1/stats_20251219_230540.csv`
- `metrics/complex1/stats_20251219_230546.csv`

Mỗi bảng được tổng hợp từ `500` lượt chạy.

### `uniform1`

| Controller | Success Rate (%) | Avg Length (px) | Avg Angle (deg) |
|---|---:|---:|---:|
| `DWAController` | 100.00 | 514.35 | 10.01 |
| `TransformerDQNController` | 100.00 | 540.76 | 18.32 |
| `DQNController` | 100.00 | 609.36 | 21.09 |
| `PER_DQNController` | 99.40 | 774.11 | 26.87 |

Nhận xét:

- `DWAController` cho đường đi ngắn nhất và mượt nhất trên map này.
- `TransformerDQNController` bám sát DWA hơn so với DQN truyền thống, nhưng vẫn kém một chút về độ ngắn và độ mượt.
- `DQNController` hoạt động ổn định nhưng đường đi dài hơn.
- `PER_DQNController` đạt tỷ lệ thành công gần như tuyệt đối nhưng đường đi trung bình dài hơn, cho thấy vẫn còn dư địa tối ưu hành vi.

### `diverse1`

| Controller | Success Rate (%) | Avg Length (px) | Avg Angle (deg) |
|---|---:|---:|---:|
| `PER_DQNController` | 98.80 | 688.47 | 30.99 |
| `TransformerDQNController` | 88.40 | 597.49 | 23.29 |
| `DQNController` | 74.20 | 671.65 | 28.71 |
| `DWAController` | 0.00 | N/A | N/A |

Nhận xét:

- `PER_DQNController` là controller ổn định nhất trên map này xét theo tỷ lệ tới đích.
- `TransformerDQNController` có đường đi ngắn hơn và góc ngoặt trung bình thấp hơn DQN và PER, nhưng tỷ lệ thành công thấp hơn PER.
- `DQNController` giảm hiệu năng rõ rệt khi môi trường đa dạng và khó hơn.
- `DWAController` không thành công trong bộ chạy hiện tại trên map này.

### `complex1`

| Controller | Success Rate (%) | Avg Length (px) | Avg Angle (deg) |
|---|---:|---:|---:|
| `TransformerDQNController` | 96.40 | 698.99 | 19.01 |
| `DQNController` | 89.00 | 719.56 | 24.01 |
| `DWAController` | 46.20 | 673.87 | 16.03 |

Nhận xét:

- `TransformerDQNController` cho tỷ lệ thành công tốt nhất trong các controller học được trên map phức tạp này.
- `DQNController` vẫn hoạt động được nhưng kém hơn về độ thành công và độ mượt.
- `DWAController` tạo ra đường đi khá mượt khi thành công, nhưng độ ổn định thấp hơn rõ rệt.

### Tóm Tắt Chung

Từ các kết quả hiện có trong repo có thể rút ra một số điểm:

- map đơn giản hơn như `uniform1` phù hợp để baseline cổ điển như `DWAController` thể hiện rất tốt
- trên map đa dạng như `diverse1`, các biến thể học sâu có ưu thế hơn về độ bền vững, đặc biệt là `PER_DQNController`
- trên map phức tạp như `complex1`, `TransformerDQNController` cho thấy sự cân bằng tốt giữa tỷ lệ thành công và chất lượng đường đi
- không có một controller nào thắng tuyệt đối ở mọi map, nên việc so sánh theo từng môi trường là rất cần thiết

Các ảnh tổng hợp và biểu đồ đã được xuất ra trong:

- `result_visualized/`
- `metrics/`

## Trực Quan Kết Quả Đường Đi

Để nhìn rõ hơn chất lượng quỹ đạo, repo đã lưu sẵn các ảnh trực quan hóa đường đi trên từng map. Mỗi map thường có 4 phiên bản:

- `best`: đường đi đại diện tốt nhất
- `success`: tỷ lệ thành công
- `length`: ưu tiên đường đi ngắn
- `angle`: ưu tiên đường đi mượt, ít góc ngoặt

### `uniform1`

![uniform1 best](result_visualized/uniform1/best_paths_uniform1_best_20251219_232255.png)

### `diverse1`

![diverse1 best](result_visualized/diverse1/best_paths_diverse1_best_20251219_232311.png)

### `complex1`

![complex1 best](result_visualized/complex1/best_paths_complex1_best_20251219_232320.png)

Nhìn tổng thể từ các ảnh này:

- đường đi trên `uniform1` khá gọn và ít vòng lặp
- `diverse1` cho thấy sự khác biệt rõ giữa các controller do vật cản đa dạng hơn
- `complex1` thể hiện bài toán khó hơn, đường đi dễ bị bẻ hướng nhiều và cần controller ổn định hơn

Nếu muốn xem thêm các góc nhìn khác của cùng một map, hãy mở các file cùng tiền tố:

- `best_paths_<map>_success_*.png`
- `best_paths_<map>_length_*.png`
- `best_paths_<map>_angle_*.png`

## Hướng Dẫn Cài Đặt

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

### 3. Cài thư viện

```bash
pip install torch torchvision torchaudio
pip install -r requirements.txt
```

Nếu máy bạn có GPU, hãy chọn bản `torch` phù hợp với CUDA đang dùng.

## Cách Chạy

### Chạy mô phỏng chính

```bash
python main.py
```

Luồng sử dụng:

1. Chọn `train` hoặc `test`
2. Chọn controller
3. Chọn map
4. Nhập số episode
5. Nếu ở chế độ test, chọn model đã lưu

### Chạy controller cổ điển

```bash
python main_classical.py
```

File này dùng cho nhóm controller baseline như `SmartAvoidanceController` và `DeterministicRRTController`.

## Output Sinh Ra

Sau mỗi lần chạy, project có thể tạo ra:

- `models/<map>/<controller>/`
  - model `.pth`
  - `best_model_*.pth`
- `plots/<map>/<controller>/`
  - biểu đồ reward theo episode
  - biểu đồ độ dài episode
- `results/<map>/<controller>/`
  - log đường đi của robot theo từng episode
- `metrics/<map>/`
  - CSV thống kê
  - bảng so sánh
  - biểu đồ so sánh
- `result_visualized/<map>/`
  - ảnh tổng hợp đường đi tốt nhất / thành công nhất / ngắn nhất / góc ngoặt thấp nhất

## Các Script Phụ Trợ

- `compare_metrics.py`: tổng hợp và so sánh metric giữa các controller
- `visualize_best_paths.py`: trực quan hóa các đường đi tốt nhất
- `MapCreator.py`: hỗ trợ tạo / chỉnh sửa map
- `MapView.py`: xem bản đồ
- `metrics.py`: tính toán các chỉ số thống kê
- `auto_test.py`, `Auto_Run.py`: tự động hóa việc chạy test

## Ghi Chú Quan Trọng

- `DWAController` chỉ chạy ở chế độ `test`
- khi test, model cần tồn tại đúng theo cặp `map/controller`
- file log đường đi được lưu riêng theo từng episode để thuận tiện phân tích lại sau này
- các kết quả trong README này phản ánh đúng các file kết quả đã có trong repo tại thời điểm cập nhật

## Cấu Trúc Thư Mục

```text
.
|-- main.py
|-- main_classical.py
|-- Robot.py
|-- Obstacle.py
|-- MapData.py
|-- Colors.py
|-- controller/
|-- utils/
|-- models/
|-- plots/
|-- results/
|-- metrics/
|-- result_visualized/
|-- map_images/
|-- README.md
```

## Kết Luận

TTTH_DQN cung cấp một môi trường đầy đủ để thử nghiệm điều hướng robot bằng học tăng cường trên lưới 2D. Điểm mạnh của project là:

- có nhiều controller để so sánh
- có sẵn map với độ khó khác nhau
- có output lưu model, đường đi và biểu đồ
- có dữ liệu thực nghiệm sẵn để đối chiếu giữa các phương pháp

Nếu bạn muốn, mình có thể viết tiếp thêm một bản README song ngữ Việt - Anh hoặc thêm phần `Troubleshooting` để người mới chạy project dễ hơn.
