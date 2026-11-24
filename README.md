# ThingSpeak와 IFTTT를 이용한 휴대폰 알림

## 1. 실습 목적

ThingSpeak 채널에 기록한 값이 특정 조건을 만족하면 IFTTT 앱으로 휴대폰 알림을 보내도록 구현하였다.

**설정 조건: Field 1에 새로 기록한 값이 30 이상이면 알림 전송**

---

## 2. IFTTT Applet 설정

IFTTT의 `Create → Classic`에서 Applet을 생성하였다.

### If This — Webhooks

- 서비스: `Webhooks`
- 조건: `Receive a web request`
- Event Name: `thingspeak_alert`

### Then That — Notifications

- 실행: `Send a notification from the IFTTT app`
- 알림 메시지:

```text
중요한 알람이 왔어용
```

저장 후 Applet이 `Connected` 상태인지 확인하였다. 휴대폰 IFTTT 앱에는 같은 계정으로 로그인하고 알림을 허용하였다.

**이미지 필요 — IFTTT Applet 설정 또는 Connected 화면**

---

## 3. ThingHTTP 설정

ThingSpeak의 `Apps → ThingHTTP → New ThingHTTP`에서 다음과 같이 설정하였다.

| 항목 | 설정 |
|---|---|
| Name | `IFTTT_Notification` |
| URL | IFTTT Applet의 Webhook URL |
| Method | `GET` |
| HTTP Version | `1.1` |

나머지 항목은 기본 상태로 두고 `Save ThingHTTP`를 눌러 저장하였다.

**이미지 필요 — ThingHTTP 설정 화면**

---

## 4. 채널 생성

`Channels → My Channels → New Channel`에서 채널을 생성하였다.

- 채널 이름: `IFTTT_Test_Channel`
- 사용할 필드: `Field 1`
- Field 1 이름: `TestValue`

Field 1을 활성화하고 `Save Channel`로 저장하였다.

---

## 5. React 조건 설정

`Apps → React → New React`에서 조건과 실행할 작업을 등록하였다.

| 항목 | 설정 |
|---|---|
| React Name | `IFTTT_React` |
| Condition Type | `Numeric` |
| Test Frequency | `On Data Insertion` |
| Channel | `IFTTT_Test_Channel` |
| Field | Field 1 — `TestValue` |
| 비교 조건 | `is greater than or equal to` |
| 기준값 | `30` |
| Action | `ThingHTTP` → `IFTTT_Notification` |
| Options | `Run action each time condition is met` |

설정 후 `Save React`로 저장하였다.

이 설정에 따라 새 데이터가 기록될 때마다 조건을 검사하고, 값이 30 이상이면 ThingHTTP를 실행한다.

**이미지 필요 — React 설정 화면**

---

## 6. URL로 데이터 기록

채널의 `API Keys → API Requests`에서 `Write a Channel Feed` URL을 복사하였다.

URL의 `field1` 값을 `40`으로 변경한 뒤 브라우저 주소창에서 실행하였다.

```text
https://api.thingspeak.com/update?api_key=YOUR_WRITE_API_KEY&field1=40
```

> `YOUR_WRITE_API_KEY`에는 해당 채널의 실제 Write API Key를 사용한다.

위 주소로 Field 1에 `40`을 기록하였다. `/update` 요청의 응답이 0보다 큰 정수이면 기록 성공이며, 이 숫자는 저장된 기록의 번호이다.

**이미지 필요 — API Requests의 기록용 URL 화면**

---

## 7. 실행 결과

Field 1에 기록한 `40`이 조건인 `30 이상`을 만족하여 다음 순서로 실행되었다.

1. React가 조건 만족을 확인한다.
2. ThingHTTP가 IFTTT Webhook을 호출한다.
3. IFTTT가 휴대폰 앱으로 알림을 보낸다.

휴대폰에서 다음 알림을 수신하여 전체 동작을 확인하였다.

```text
ThingSpeak에서 설정한 조건이 만족되었습니다.
```

**이미지 필요 — 최종 수신한 휴대폰 알림 화면**

---

## 주의 사항

- GitHub에 실제 API Key나 Webhook 개인 키를 공개하지 않는다.
- 캡처 이미지에서도 키가 보이는 부분은 가린다.
- 현재 설정은 새로 기록한 값이 30 이상일 때마다 알림을 요청한다.
