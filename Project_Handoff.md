# Project_Handoff.md - esp-wifi-connect / Xiaozhi Bank

> **Status:** v1.1.1 CAPTIVE JSON/BANK SAVE FIX IMPLEMENTED - STATIC 7/7 PASS - USER BUILD/FIELD PENDING

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
- `idf_component.yml`: component version `1.1.1`.

Security contract:

- GET never serializes plaintext `sepay_key`.
- `****` Save = unchanged.
- blank Save = erase NVS key.
- new text = replace.
- secret value is never printed in Bank save log.

Build note: publish/tag this repo as `v1.1.1` before clean-building OTA_V1 `2.0.5.30`; firmware manifest now requires that tag.

## v1.1.1 - 2026-11-04

USER field screenshot showed GET `/bank/config` working but Save Bank reporting browser `Failed to fetch`.

RCA: v1.1.0 called `httpd_req_recv()` only once for JSON POST bodies. HTTP/TCP may split a body across receives, so partial JSON could be parsed and the handler could close with `ESP_FAIL`.

Implemented:

- shared bounded `ReadRequestBody()` loops until full `content_len` is received; up to 3 receive timeouts are tolerated;
- `/submit`, `/bank/submit`, `/advanced/submit` all use the full-body pattern;
- Bank errors send a complete HTTP response and return `ESP_OK`;
- Bank save logs only state/reason through `[BANK_CONFIG_AP]`, never plaintext secret;
- browser retries Bank Save exactly once only when `fetch()` itself fails at network level; normal HTTP 4xx/5xx responses are not blindly retried;
- `preflight_esp_wifi_connect.py` guards this contract.

Security semantics are unchanged: `****`=keep existing key, blank=clear, new text=replace; plaintext SePay key is never returned or logged.
