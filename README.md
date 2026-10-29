# plugin-builder-ops

Plugin Builder 설치 앱의 상태 확인 2순위 파일을 두는 공개 저장소다. `kh1012/atelier-ops` 와 같은 꼴에 "가장 낮은 허용 버전"(`minVersion`)을 더했다.

앱은 먼저 사내 배포 서버(`POST /health`)에 묻는다. 서버가 답하면 서버가 정한다. 거기에 못 닿을 때만 이 저장소의 `gate.json` 을 읽는다. 배포 PC 가 사내망 밖에 있거나 꺼져 있어도, 그리고 설치 파일이 사내망 밖으로 새었을 때도 밖에서 앱을 끌 수 있게 하려고 둔다.

## gate.json

| 칸                    | 뜻                                                                                                                   |
| --------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `gate`                | 차단 스위치. `false` 면 어떤 경우에도 막지 않는다(점검 · 유예 · `minVersion` 모두 쓰지 않는다)                        |
| `maintenance.on`      | `gate` 가 `true` 일 때 점검으로 막는다                                                                               |
| `maintenance.message` | 점검으로 막을 때 앱에 띄울 사유                                                                                      |
| `graceHours`          | `gate` 가 `true` 일 때, 마지막 연결 뒤 이 시간이 지나면 막는다. 1 ~ 336 사이의 정수                                   |
| `minVersion.dv`       | `gate` 가 `true` 일 때, dv 채널 앱의 버전이 이 값보다 낮으면 막는다. `null` 이면 버전으로 막지 않는다                  |
| `minVersion.pr`       | 같은 규칙으로 Production 채널. 채널을 가르는 것은 Production 이 dv 보다 늦게 올라가서다(dv 를 올려도 pr 은 그대로)   |
| `updatedAt` · `note`  | 판정에 안 쓴다                                                                                                       |

모양이 틀린 파일(예: `gate` 가 참 · 거짓이 아님, `minVersion` 이 `x.y.z` 꼴이 아님)은 못 읽은 것과 같게 본다. 그 파일로는 막지도 풀지도 않는다.

## 설치 파일이 새었을 때

1. 새 패치를 dv 로 배포한다(`pnpm builder:version` → `pnpm builder:deploy`). 사내망의 정상 사용자는 앱 안 업데이트로 넘어간다
2. `gate` 를 `true` 로, 새어 나간 채널의 `minVersion` 을 새 패치 버전으로 올린다. 새 사본은 업데이트를 받지 못해 그 버전에 묶여 막힌다
3. 정상 기기가 새 버전으로 옮겼는지 배포 PC 의 현황판으로 본다
4. 급하면 `maintenance.on` 을 `true` 로 해 모두 막는다

## 고치는 법

- 배포 PC 에서 `pnpm builder:ops gate on|off` · `maintenance on "사유"|off` · `grace <시간>` · `min <채널> <버전>|off` 를 하면 이 파일도 함께 고쳐진다(M9 에서 짓는다). `pnpm builder:ops mirror` 로 올라간 내용이 지금 상태와 같은지 본다
- github.com 에서 손으로 고쳐도 된다. `{"gate": false}` 처럼 줄여 써도 읽는다
- raw 는 CDN 이 몇 분 묵힌다. 앱의 5분 주기와 합쳐 고친 값이 앱에 닿기까지 10분쯤 걸릴 수 있다

## 알아 둘 것

- 서명이 없다. 이 저장소의 쓰기 권한이 믿음의 근거다
- 공개 저장소다(2026-10-29 비공개로 만든 뒤 같은 날 공개로 바꿨다). 앱이 토큰 없이 raw 를 받아야 해서다. 이 파일에 비밀을 넣지 않는다. 앱에 토큰을 담는 길은 쓰지 않는다(토큰이 만료 · 폐기되면 사내망 밖 앱이 모두 막히고, 설치판에서 꺼낼 수 있다)
- 막는 것은 앱 실행뿐이다. 설치판 안의 코드는 꺼낼 수 있다
- 앱이 이 파일을 읽는 것은 M9 부터다
- 설계: plugin-builder `apps/builder/docs/2-plan/roadmap.md` 의 M9 줄과 배포 계획 `apps/builder/docs/2-plan/core/release-plan.md`. 바탕은 atelier `apps/atelier/docs/2026-09-30-health-gate-design.md`
