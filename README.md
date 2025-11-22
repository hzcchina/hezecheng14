
ADB 设备连接状态监控工具
import subprocess
import time
import json
from datetime import datetime
class AdbDeviceMonitor:
def __init__(self):
self.device_history = {}
self.log_file = "adb_device_log.json"
self.load_history()
def load_history(self):
"""加载设备连接历史记录"""
try:
withopen(self.logfile,'r')asf:
self.device_history = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
self.device_history = {"devices": {}}
def save_history(self):
"""保存设备连接历史记录"""
withopen(self.logfile,'w')asf:
json.dump(self.device_history, f, indent=2)
def get_connected_devices(self):
"""获取当前连接的设备列表"""
result = subprocess.run(
["adb", "devices"],
capture_output=True,
text=True
)
output = result.stdout.strip()
devices = []
for line in output.split('\n')[1:]:
if line.strip() and 'device' in line:
device_id = line.split()[0]
devices.append(deviceid)return devices
def get_device_info(self,deviceid):
"""获取设备详细信息"""
info_commands = {
"model": ["adb", "-s", device_id, "shell", "getprop", "ro.product.model"],
"manufacturer": ["adb", "-s", device_id, "shell", "getprop", "ro.product.manufacturer"],
"android_version": ["adb", "-s", device_id, "shell", "getprop", "ro.build.version.release"],
"serial": ["adb", "-s", device_id, "shell", "getprop", "ro.serialno"]
}
info = {}
for key, cmd in info_commands.items():
try:
result = subprocess.run(
cmd,
capture_output=True,
text=True,
timeout=5
)
info[key] = result.stdout.strip()
except Exception as e:
info[key] = f"获取失败: {str(e)}"
return info
def monitor_devices(self, interval=5):
"""持续监控设备连接状态"""
print("开始监控ADB设备连接状态 (按Ctrl+C停止)...")
try:
while True:
current_devices = self.get_connected_devices()
timestamp = datetime.now().isoformat()
# 检查新连接设备
for device_id in current_devices:
if device_id not in self.device_history["devices"]:
print(f"\n[+] 新设备连接: {device_id}")
device_info = self.get_device_info(deviceid)self.device_history["devices"]deviceid = {
"first_connected": timestamp,
"last_seen": timestamp,
"info": device_info,
"connection_log": [{"status": "connected", "time": timestamp}]
}
print(f"设备信息: {json.dumps(device_info, indent=2)}")
self.save_history()
else:
# 更新最后 seen 时间
self.device_history["devices"]deviceid]["lastseen" = timestamp
# 检查断开连接设备
for device_id in list(self.device_history["devices"].keys()):
if device_id not in current_devices:
print(f"\n[-] 设备断开连接: {device_id}")
self.device_history["devices"]deviceid]["connectionlog".append({
"status": "disconnected",
"time": timestamp
})
self.save_history()
# 可以选择保留历史记录或移除
# del self.device_history["devices"]deviceid
time.sleep(interval)
except KeyboardInterrupt:
print("\n监控已停止")
self.save_history()
if __name__ == "__main__":
monitor = AdbDeviceMonitor()
monitor.monitor_devices()
使用说明
1.确保已安装ADB并配置环境变量
2.运行脚本前连接Android设备并开启USB调试
3.支持功能:
o实时监控设备连接/断开状态
o自动记录设备详细信息(型号、厂商、Android版本等)
oJSON格式保存连接历史记录
o可配置监控间隔时间
依赖
Python 3.6+
ADB工具
适当的设备权限(USB调试已开启)
此工具会在当前目录生成adb_device_log.json文件，记录所有连接过的设备信息和连接状态变化历史。代码中已包含错误处理和超时控制，可直接用于日常ADB设备管理和测试场景。
