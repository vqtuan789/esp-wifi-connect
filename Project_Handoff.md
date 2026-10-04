# Project_Handoff.md - esp-wifi-connect / Xiaozhi Bank

> **Status:** PHASE 1.0 BANK TAB IMPLEMENTED - BUILD/HARDWARE PENDING

## Vai trò

Thêm tab `🏦 Bank` vào captive WiFiConfig, không đưa runtime SePay/WSS nặng vào HTTP handler.

## Fields

| Field | Rule |
|---|---|
| Bank Device ID | read-only, 12 HEX uppercase |
| SePay API Key | write-only secret; `****`=đã có/giữ nguyên; user xóa mask thành blank rồi Save = clear; new text=replace |
| EC | default 50 |
| Serial Log | Enable/Disable, dùng cùng persistent Bank setting với WebUI |
| WSS | blank = firmware defaults/failover |
| NAT URL | full `https://domain:port/sepay`, blank valid; chỉ là khai báo NAT, **không khóa WebServer `/sepay`** |

## Security

Không copy pattern Weather API key đang GET/log plaintext. Bank key phải theo LoaBank: không trả plaintext, không log, constant-time compare tại webhook/auth path nơi phù hợp.

## Interaction

Portal chỉ validate/save config. SePay key UX: `****` giữ nguyên, blank sau khi xóa mask = clear, text mới = replace; không cần action Clear riêng. Sau save, BankService reload/apply theo contract firmware; không chạy blocking SePay/WSS request trong form handler.


## Phase 1 implementation

Changed component source:

- `assets/wifi_configuration.html`: new `🏦 Bank` tab and masked-secret UX.
- `wifi_configuration_ap.cc`: `/bank/config`, `/bank/submit`, separate `bank` NVS, validation, Serial log toggle.
- `idf_component.yml`: component version `1.1.0`.

Security contract:

- GET never serializes plaintext `sepay_key`.
- `****` Save = unchanged.
- blank Save = erase NVS key.
- new text = replace.
- secret value is never printed in Bank save log.

Build note: publish/tag this repo as `v1.1.0` before building OTA_V1 `2.0.5.07`.
