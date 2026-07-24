# HostEye Endpoint Agent

Bộ cài Windows thu thập Windows Event Logs và Sysmon, sau đó gửi telemetry trực tiếp tới Docker HostEye trên máy nhận qua Radmin VPN:

```text
http://26.117.73.174:8080/api/v1/telemetry
```

## Yêu cầu

- Windows 10/11 hoặc Windows Server 64-bit.
- Tài khoản có quyền Administrator.
- Máy nhận chạy Docker Desktop, container `hosteye-all-in-one-local` và Radmin VPN với địa chỉ `26.117.73.174`.
- Máy gửi cài Radmin VPN và tham gia cùng mạng riêng với máy nhận.
- Máy gửi truy cập được TCP `26.117.73.174:8080`.

## Cài đặt

1. Tải `HostEye_Endpoint_Installer.exe`.
2. Kết nối máy gửi vào cùng mạng Radmin VPN với máy nhận.
3. Kiểm tra kết nối:

   ```powershell
   Test-NetConnection 26.117.73.174 -Port 8080
   Invoke-RestMethod "http://26.117.73.174:8080/api/health"
   ```

4. Nhấn phải vào file và chọn **Run as administrator**.
5. Nếu Windows SmartScreen cảnh báo vì file chưa được ký số, chọn **More info** rồi **Run anyway** sau khi kiểm tra SHA-256.
6. Chờ trình cài đặt hoàn tất. Agent, Sysmon và shipper sẽ được khởi động tự động.

SHA-256:

```text
5A0B1A119231C16D3D0092997FC44BD09347518107712AA4EB67ACA4AB4D1A37
```

Kiểm tra hash:

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
- Log shipper không có lỗi kết nối lặp lại.

## Máy nhận

Khởi động container nếu cần:

```powershell
docker start hosteye-all-in-one-local
```

Kiểm tra gateway:

```powershell
Invoke-RestMethod "http://localhost:8080/api/health"
docker ps --filter name=hosteye-all-in-one-local
```

Dashboard:

```text
http://localhost:8080/dashboard/
```

## Lưu ý

- `26.117.73.174` là địa chỉ Radmin VPN của máy nhận, không phải domain hoặc public IP của router.
- Không forward trực tiếp cổng `8080` trên router. Kết nối này được thiết kế để chạy trong mạng Radmin VPN riêng.
- Bộ cài hiện chưa có chữ ký code-signing.
- Chỉ triển khai agent trên máy bạn sở hữu hoặc được phép quản trị.
