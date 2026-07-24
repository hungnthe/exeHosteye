# HostEye Endpoint Agent

Bộ cài Windows thu thập Windows Event Logs và Sysmon, sau đó gửi telemetry tới:

```text
https://hosteye.iahn.hanoi.vn/api/v1/telemetry
```

## Yêu cầu

- Windows 10/11 hoặc Windows Server 64-bit.
- Tài khoản có quyền Administrator.
- Máy có kết nối Internet tới `hosteye.iahn.hanoi.vn` qua HTTPS cổng 443.

## Cài đặt

1. Tải `HostEye_Endpoint_Installer.exe`.
2. Nhấn phải vào file và chọn **Run as administrator**.
3. Nếu Windows SmartScreen cảnh báo vì file chưa được ký số, chọn **More info** rồi **Run anyway** sau khi đã kiểm tra SHA-256.
4. Chờ trình cài đặt hoàn tất. Agent, Sysmon và shipper sẽ được khởi động tự động.

SHA-256 của bản phát hành này:

```text
F9F579B9FD09D961E9BC81AFC4345902C028CE0C2ECD93221420033402EFEDC6
```

Kiểm tra hash sau khi tải:

```powershell
Get-FileHash .\HostEye_Endpoint_Installer.exe -Algorithm SHA256
```

## Kiểm tra agent

Mở PowerShell bằng quyền Administrator:

```powershell
Get-Service HostEyeAgent
Get-ScheduledTask -TaskName HostEye_Shipper_Startup
Get-Content "C:\ProgramData\HostEye\logs\shipper.log" -Tail 50
```

Trạng thái mong đợi:

- Service `HostEyeAgent` đang chạy.
- Scheduled task `HostEye_Shipper_Startup` tồn tại và đang hoạt động.
- Log của shipper không có lỗi kết nối lặp lại.

Kiểm tra gateway:

```powershell
Invoke-RestMethod "https://hosteye.iahn.hanoi.vn/api/health"
```

Dashboard:

```text
https://hosteye.iahn.hanoi.vn/dashboard/
```

## Lưu ý

- Không thêm dòng `127.0.0.1 hosteye.iahn.hanoi.vn` vào file `hosts` trên máy cài agent.
- Bộ cài hiện chưa có chữ ký code-signing.
- Chỉ triển khai agent trên máy bạn sở hữu hoặc được phép quản trị.
