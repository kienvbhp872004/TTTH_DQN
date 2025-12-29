
---

## 📦 Clone dự án

```bash
git clone https://github.com/jien0404/TTTH_DQN.git
cd TTTH_DQN
```
## Tạo môi trường ảo
```
python -m venv .venv
```
# Kích hoạt:
# Windows:
```
.venv\Scripts\activate
```
# macOS/Linux:
```
source .venv/bin/activate
```

# Cài đặt thư viện cần thiết
```
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install -r requirements.txt
```
#Cấu trúc thư mục
```
├── Models                 # Chứa pre-train model
├── results                # 
├── plots                  # 
├── main.py                # File chạy chính
├── Robot.py               # Định nghĩa lớp Robot
├── Controller.py          # Bộ điều khiển Q-learning
├── Colors.py              # Định nghĩa màu sắc
├── Obstacles.py           # Định nghĩa chướng ngại vật
├── MapData.py             # Thông tin về bản đồ
├── README.md              # File mô tả (chính là file này)
```

#Chạy chương trình
```
python main.py
```
