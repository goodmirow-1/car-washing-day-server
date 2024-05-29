# Concert Ticketing Server

TDD 기반 콘서트 티케팅 서버 구축 프로젝트 입니다.

# 목차

-   [요약](#요약)
-   [주요 기능](#주요기능)
-   [트러블 슈팅](#트러블슈팅)
    -   [서로 다른 데이터를 제공하는 (단기,중기) 기상 예보](#서로-다른-데이터를-제공하는-(단기,중기)-기상-예보)
    -   [공공 API 요청 실패 시 대응](#공공-API-요청-실패시-대응)

# 요약

- 사용자는 앱에서 제공하는 기상 예보 데이터를 통해 세차 지속일을 확인합니다.
- 원하는 날짜를 세차일로 등록하며, 세차일에 기상 상황에 따라 FCM을 보냅니다.

# 주요기능

- 회원가입
    - 카카오, 애플, 구글 및 비회원으로 로그인이 가능해야 함
    - 사용자는 세차일 추천 알고리즘에 사용되는 강수확률을 정할 수 있어야 함
- 기상 예보 공공API를 활용한 데이터 수집
    - 기상 관련 공공API를 사용하여 단기(0~2일) 및 중기(3~10일) 예보 데이터를 가져와야 함
    - 서로 다른 데이터를 제공하는 각각의 기상 예보 API 결과값을 클라이언트에게 동일한 정보를 제공할 수 있도록 정형화해야 함
    - 공공API 요청은 하루에 제한되어 있으며, 성능을 고려하여 기상 예보 API 요청을 최소화해야 함
    - 기상 예보 API 요청이 실패할 경우 대응 방안 필요
- 세차일 등록 및 알람
    - 사용자는 클라이언트를 통해 원하는 날짜의 세차일을 등록할 수 있어야 함
    - 사용자가 설정한 강수확률을 기반으로 세차 당일 날씨 예보에 따라 FCM을 보내줘야 함

# 트러블슈팅

## 서로 다른 데이터를 제공하는 (단기,중기) 기상 예보

**문제:** 공공API 포탈 사이트에서 제공하는 단기 예보와 중기 예보의 response 값이 다릅니다. 세차 지속일을 계산하기 위해서는 “강수량”이 필요하고, 예보를 위해서는 “강수 형태” 데이터가 필요한데, 단기 예보는 매시간마다 데이터 수집이 가능하지만 중기 예보는 오전/오후 데이터만 수집 가능합니다.

**해결:**

- 클라이언트와 논의하여 다음과 같은 데이터로 가공해서 제공하기로 했습니다.
    1. 단기와 중기에 해당하는 각각의 API를 제공한다.
    2. 중기의 경우 단기의 3일치 데이터를 중기처럼 오전, 오후로 가공해서 제공한다.
    3. 사용자가 조회한 시간에 가장 가까운 다음 시간의 단기 데이터 강수량 및 강수 형태 데이터를 추가로 제공한다.
## 공공 API 요청 실패시 대응

**문제:** 어플리케이션 특성상 기상 예보 데이터는 반드시 조회되어야 합니다. 여러 데이터를 한 번에 API로 요청할 경우, 정상적으로 응답받지 못하는 경우가 발생할 수 있습니다.

**해결:**

- 정상적으로 응답받지 못한 데이터를 list로 만들어서, API를 다시 요청한다.

**개선 방안:**

- 해당 공공 데이터 조회가 불가능할 경우를 대비하여, 요청 실패 시 이벤트를 발생시켜 N초마다 조회를 요청하고, 일정 수/시간 동안 실패할 경우 서버를 내리거나 로그를 남겨야 할 것 입니다.
