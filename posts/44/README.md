# Github Action 맛보고, AWS S3에 Vue 자동으로 배포하기

# \# 들어가며

정해진 [배포 파이프라인](https://en.wikipedia.org/wiki/Continuous_integration#CI/CD)이 없는 프로젝트에서 `배포`는 귀찮지만, 반드시 해야 하는 `구몬 숙제` 같은 느낌입니다. 아무리 열심히 + 잘 + 완벽한 프로젝트를 만들어도, `배포`하여 세상에 공개하지 않으면 아무런 의미가 없죠 (만약 그 프로젝트가 예제 책에 나올만한 `계산기` 예제가 아닌, "널리 세상을 이롭게" 까진 아니라도, 누군가 사용하고 고쳐지고 발전할 소중한 프로젝트라면 말이죠).

"통합과 배포.. 어떤 방법이 최선인가"라는 고민을 하고 있고, 뭔가 "코드 자체를 대부분 `github` 에 올리니까, `github action` 자체는 꽤 괜찮은 배포 수단이 될 수 있지 않을까?" 하는 생각과 관심이 최근에 공개된 `Github Action`에 생겼고, "어? 어차피 사이드 프로젝트 하는 김에 아예 사용해 볼까?" 하는 생각을 했습니다. 그래서 써보게 되었고, 그 과정과 방법을 정리 & 소개합니다.

# \# 뭘 만드는 건가요?

먼저 `React`, `Vue` 같은 [single-page application (SPA)](https://en.wikipedia.org/wiki/Single-page_application) 은 소스를 통합하는 과정을 거쳐서 `HTML`, `CSS`, `Javascript`를 해석 가능한 환경에 `배포` 합니다. 물론 지금 언급한 코드만 배포 과정이 필요 한 것이 아니라, 거의 모든 코드와 서비스가 [지속적 통합(CI)](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81_%ED%86%B5%ED%95%A9)과 [지속적인 배포(CD)](https://ko.wikipedia.org/wiki/%EC%A7%80%EC%86%8D%EC%A0%81_%EB%B0%B0%ED%8F%AC) 라는 주제로 고민합니다. 제 필요성에 의해 `AWS S3`에 `Vue` 코드를 배포 할 필요성이 생겼고, 저는 여기서 `Vue` 로 작성된 프론트엔드 코드를 `github` 에 올리는 행위만으로, 자동으로 제가 지정한 위치에 코드가 자동으로 업로드되고 작동되도록 만들려고 합니다.

![https://ehq7xg.sn.files.1drv.com/y4mRIRWaz92fs85IYMZlIc0KuaJfaLhIz0yOxbYx3ISC8c3-98RKUj5AEm6m8tU4y4IBHXlvTpHBX7KRRZrMDS9BG0svUx1o4RKOMz_XfYX31acV7iBNAajohJFjw02FV6vA5Zd5idn6dN8wUZ9bmZ4RlTiEjT6eCy32Umwiukxud-5sAtLX1I3Bf4pxQjc5VfbB8sG4ptaWQgO6i-vKm2eoQ?width=853&height=366&cropmode=none](https://ehq7xg.sn.files.1drv.com/y4mRIRWaz92fs85IYMZlIc0KuaJfaLhIz0yOxbYx3ISC8c3-98RKUj5AEm6m8tU4y4IBHXlvTpHBX7KRRZrMDS9BG0svUx1o4RKOMz_XfYX31acV7iBNAajohJFjw02FV6vA5Zd5idn6dN8wUZ9bmZ4RlTiEjT6eCy32Umwiukxud-5sAtLX1I3Bf4pxQjc5VfbB8sG4ptaWQgO6i-vKm2eoQ?width=853&height=366&cropmode=none)

궁극적으로 `자동 배포`라는 행위 자체가, 제가 "코드를 빌드(통합)하고 → 빌드된 결과물을 옮기고 → 배포 환경에서의 정상 동작을 확인" 하는 일련의 과정에서 사용되는 아까운 제 노동력을 더 효율적인 곳에 사용 할 수 있게 하는 것(=코드의 생산성 자체에 더 집중하는 것) + 지정된 행동을 반복해야 하는 것이라면, 실수가 발생할 수 있는 `인력` 보단 `컴퓨팅` 에 맡기는 것 (그리고, "생각"이 필요 없는 이런 반복된 동작은 똑똑한 "사람" 보다, 멍청한 "컴퓨터" 가 더 잘합니다..) 인 것 같습니다.

# \# 준비물

-   소스 코드 업로드를 위한 [github](https://github.com/) 계정 → 없을 경우 해당 [포스트](https://bin-e.tistory.com/24)로
-   프로젝트 코드를 [AWS S3](https://aws.amazon.com/ko/s3/)에 배포하기 위한 [AWS](https://aws.amazon.com/ko/) 계정 → 없을 경우 해당 [포스트](https://bin-e.tistory.com/21)로
-   S3를 통하여 배포 할 소스 코드(이 프로젝트는 [Vue.js](https://vuejs.org/)) → 다른 언어여도 상관은 없음

# \# 본격적으로 만들어 봅시다

## github에 사용할 코드 올리기

이 과정은 굳이 `vue` 가 아니어도 상관은 없고, 기존에 `github` 에 올라가 있는 소스를 그대로 사용한다면 건너뛰어도 괜찮습니다. 단, 다른 코드를 사용한다면 빌드 과정이 다를 수 있으니, 하단의 REFERENCES에서 공식 다큐먼트를 확인하여 본인에게 맞는 배포환경을 만드시길 바랍니다.

### vue 프로젝트 생성

```
$ vue -V # vue-cli 버전이 2.x 이하라면, vue-cli를 통한 프로젝트 생성 방법이 다르기 때문에 확인하며, 2.x 버전의 경우 바로 아래의 링크를 참고하세요.
@vue/cli 4.1.2 
$ vue create vue-github-action # 형식 : vue create <APP_NAME>
$ # 생성 과정이 있지만, 간단하기에 설명하지 않습니다. 궁금하신 분은 아래의 공식문서 를 참고 합니다.
$ cd vue-github-action
$ npm install
$ npm run serve # 정상 동작을 확인합니다. (exit = Ctrl + c)
```

-   [vue-cli 2.x에서 vue create 시작하기](https://cli.vuejs.org/guide/creating-a-project.html#pulling-2-x-templates-legacy)
-   [공식문서](https://cli.vuejs.org/guide/creating-a-project.html)

![https://ehqqxg.sn.files.1drv.com/y4mIPtEqxDNb7wkDg7RWdkoQHa2kHbaXBvWxhSX_z_HtoS-iIpPsOCMUgIP7QmqIuS9xpE6jq649hiXP_UmVOQ_XTiyzaGXhr9I4xbS--A-o27F9riXQkmWuhL8GlPgPtkiuztIrdI-VjskkHp4ZSIPmh2tgQlN5vTL0Fb_Deb4Um9x8J6aOAP3y1F176W0O8q-UmoMXwiU7CjsqmqYz-gE5w?width=383&height=197&cropmode=none](https://ehqqxg.sn.files.1drv.com/y4mIPtEqxDNb7wkDg7RWdkoQHa2kHbaXBvWxhSX_z_HtoS-iIpPsOCMUgIP7QmqIuS9xpE6jq649hiXP_UmVOQ_XTiyzaGXhr9I4xbS--A-o27F9riXQkmWuhL8GlPgPtkiuztIrdI-VjskkHp4ZSIPmh2tgQlN5vTL0Fb_Deb4Um9x8J6aOAP3y1F176W0O8q-UmoMXwiU7CjsqmqYz-gE5w?width=383&height=197&cropmode=none)

-   기본 프로젝트 구성은 위 사진과 같을 것 입니다.

### 새 github repository 만들기

![https://fbq7xg.sn.files.1drv.com/y4mgaMLwoJG6Q36MN34zd1tU_DkrTL0BkJngK3qwgikSWRq0-z_ze7A51zFbnazjVhMEIyA71733aaoLZVzVQOy5Kh9My5Urrhaq5JV_S2s8yqWU8dzoYayxgJCe9glARLDS0Dr7QetwHAi8g6rgNMDtK6hlSNIDzapJxXZIovkC_sm4fSo7BQELPTaAV4diOb_wVssQtWNwU8PFz7hZugukg?width=341&height=350&cropmode=none](https://fbq7xg.sn.files.1drv.com/y4mgaMLwoJG6Q36MN34zd1tU_DkrTL0BkJngK3qwgikSWRq0-z_ze7A51zFbnazjVhMEIyA71733aaoLZVzVQOy5Kh9My5Urrhaq5JV_S2s8yqWU8dzoYayxgJCe9glARLDS0Dr7QetwHAi8g6rgNMDtK6hlSNIDzapJxXZIovkC_sm4fSo7BQELPTaAV4diOb_wVssQtWNwU8PFz7hZugukg?width=341&height=350&cropmode=none)

-   새로운 `repository` 를 생성 합니다.

![https://exq8xg.sn.files.1drv.com/y4m18BcBHQY_n2L4HDWm5QcR6VvrS8H0uPyjNwtwccxOTT0fl0b6KSBXcw2uuac5FvtHeaDURomIfvuztzUE6LLe5AUcpYaWkChXrv6s7ebJGNw8ftMeSVFHH0CZLSYGyYY2Pl2g_BYuL9Xjpl4sLLAZFkhhwmo4MjsZAV3HLI-hf6k56SZJV_Lyxd5Zkh76GG7_GIR0sE5YJ94_vY_0UMomw?width=726&height=660&cropmode=none](https://exq8xg.sn.files.1drv.com/y4m18BcBHQY_n2L4HDWm5QcR6VvrS8H0uPyjNwtwccxOTT0fl0b6KSBXcw2uuac5FvtHeaDURomIfvuztzUE6LLe5AUcpYaWkChXrv6s7ebJGNw8ftMeSVFHH0CZLSYGyYY2Pl2g_BYuL9Xjpl4sLLAZFkhhwmo4MjsZAV3HLI-hf6k56SZJV_Lyxd5Zkh76GG7_GIR0sE5YJ94_vY_0UMomw?width=726&height=660&cropmode=none)

-   `Create a new repository` 의 과정에 따라 내용을 작성하고 `Create repository` 합니다.

![https://exrgxg.sn.files.1drv.com/y4mUc9IGHzOfyqsIVCvub8akmeyexr8eHHkTT0H7HuxSk_CG-bLBe_dalK-0M-04ybN0k74GyaKnOTzU4OqPHJYN630NuYe1mn_fHpCfzGzMBnBsJIPjCFEUsXJrTSfwad5QaSnVG8Swl5oUWF4V_GCf5tfrd4pQ50ELkY4MYA8BYrmeoc6iAOeHm7M3zsYiVIkjhZBjLghHIaMD7w0l7HS7Q?width=988&height=811&cropmode=none](https://exrgxg.sn.files.1drv.com/y4mUc9IGHzOfyqsIVCvub8akmeyexr8eHHkTT0H7HuxSk_CG-bLBe_dalK-0M-04ybN0k74GyaKnOTzU4OqPHJYN630NuYe1mn_fHpCfzGzMBnBsJIPjCFEUsXJrTSfwad5QaSnVG8Swl5oUWF4V_GCf5tfrd4pQ50ELkY4MYA8BYrmeoc6iAOeHm7M3zsYiVIkjhZBjLghHIaMD7w0l7HS7Q?width=988&height=811&cropmode=none)

-   생성된 `repository` 의 정보를 확인하고, 주소를 복사합니다 (붉은 박스 오른쪽 끝에 버튼을 클릭).

### `github` 의 `repository` 에 소스 업로드 하기

```
$ pwd # 작업은 방금 위에서 만들었던 프로젝트 폴더에서 할 것 입니다.
/d/_Projects/vue-github-action 
$ git remote -v
$ # 아무것도 출력되는 것이 없어야 현재 연결된 repository가 없는 것 입니다.
$ git remote add origin https://github.com/bin-e/vue-github-action.git
$ git remote -v
origin  https://github.com/bin-e/vue-github-action.git (fetch)     
origin  https://github.com/bin-e/vue-github-action.git (push) # 이렇게 연결된 repo가 출력 됩니다.
$ # 원래는 git init 과정과, git add, git commit 과정을 진행해야 하지만, vue create로 생성한 프로젝트는 기본으로 init commit이 된 상태로 생성 됩니다.
$ # 따라서 바로 push 합니다.  
$ git push origin master
```

![https://ehq6xg.sn.files.1drv.com/y4mevWqNMBRKtlcEUBL1tTIVUeJHskgoYTj5qNbGJVrm9WiGicG18n7onYSLClMkO-YMx2nWYL8ef9KRd_zpln59idysPjicsJmSWriuhbNnRewdzd0sn0QThGjkcO3BNEF2BnZHKnLtWoA-tBkksCLjkvVf0PI0fpmJkjPt68OHHlZdEG4DHzFWjcx9RQkmkzCI66zZiClwTDpt_0ceZw-JA?width=999&height=900&cropmode=none](https://ehq6xg.sn.files.1drv.com/y4mevWqNMBRKtlcEUBL1tTIVUeJHskgoYTj5qNbGJVrm9WiGicG18n7onYSLClMkO-YMx2nWYL8ef9KRd_zpln59idysPjicsJmSWriuhbNnRewdzd0sn0QThGjkcO3BNEF2BnZHKnLtWoA-tBkksCLjkvVf0PI0fpmJkjPt68OHHlZdEG4DHzFWjcx9RQkmkzCI66zZiClwTDpt_0ceZw-JA?width=999&height=900&cropmode=none)

-   코드가 정상적으로 잘 올라갔는지 확인 합니다.

## AWS S3 셋팅하기

![https://ebqqxg.sn.files.1drv.com/y4meJwy18uN8pKmtzbxjLviBVfBZYOOCPxQRDJGy7isS6wFB66LISmjw9PoyokHi7ceb-oU_iVhYxWS44E_egcgrs9UEav8PJfsf06lj6PL9vN7pKJUPSRPadNbhoyDcRw56P4_bdNs-ZTuHd041Tasw76-PUT-7DQ8LCjH6vCjnlrMGVsM5ZH0eMBBa5viDrpi4-jyIjeNzGqkOtAeVme_jg?width=1252&height=506&cropmode=none](https://ebqqxg.sn.files.1drv.com/y4meJwy18uN8pKmtzbxjLviBVfBZYOOCPxQRDJGy7isS6wFB66LISmjw9PoyokHi7ceb-oU_iVhYxWS44E_egcgrs9UEav8PJfsf06lj6PL9vN7pKJUPSRPadNbhoyDcRw56P4_bdNs-ZTuHd041Tasw76-PUT-7DQ8LCjH6vCjnlrMGVsM5ZH0eMBBa5viDrpi4-jyIjeNzGqkOtAeVme_jg?width=1252&height=506&cropmode=none)

-   [AWS console](https://console.aws.amazon.com/console/home) 에 로그인하여 서비스에 `S3` 를 검색합니다.

![https://dxqqxg.sn.files.1drv.com/y4m869BLPNhehkaNNNvJ62oHo794wtGHIOd2geCRIsSXQYM0-DHBN01wEbjWbScpa_0mLci0yrZU1Zlbcnq1tUIsqz4_5DiMO7LtERtr0lX6VPyjo5vAmysuqmYq9fdTiNRxhPxoctNN6dNweELU-7nXwGaz5Oq_EG_IzB7aeBT9LWL1PBUfi_FibNiDjITL7M4IUEksSAe0172IU3BK4P3jQ?width=921&height=608&cropmode=none](https://dxqqxg.sn.files.1drv.com/y4m869BLPNhehkaNNNvJ62oHo794wtGHIOd2geCRIsSXQYM0-DHBN01wEbjWbScpa_0mLci0yrZU1Zlbcnq1tUIsqz4_5DiMO7LtERtr0lX6VPyjo5vAmysuqmYq9fdTiNRxhPxoctNN6dNweELU-7nXwGaz5Oq_EG_IzB7aeBT9LWL1PBUfi_FibNiDjITL7M4IUEksSAe0172IU3BK4P3jQ?width=921&height=608&cropmode=none)

-   `버킷 만들기`를 클릭

![https://dxq6xg.sn.files.1drv.com/y4mCKJMzzKccRoX2tKkNC3MHG3bHifgG33Cc1vyB9mMDu6VtCva708YnFpzx0J0bvRyzk2UVbLsuzrUx_SdamAbMfcWP4Wf_nTnIrBuJTQr62o0Z4y0lWVCwq6LfCz8eW3J0mnJznDVge8F8cPOxk1lL33wDU69x_nEtdxrEweXbXDjcjGrNpawL3fcq4Oy4RcTiTRBJSK-LQLqxE0gRk3uTw?width=1110&height=816&cropmode=none](https://dxq6xg.sn.files.1drv.com/y4mCKJMzzKccRoX2tKkNC3MHG3bHifgG33Cc1vyB9mMDu6VtCva708YnFpzx0J0bvRyzk2UVbLsuzrUx_SdamAbMfcWP4Wf_nTnIrBuJTQr62o0Z4y0lWVCwq6LfCz8eW3J0mnJznDVge8F8cPOxk1lL33wDU69x_nEtdxrEweXbXDjcjGrNpawL3fcq4Oy4RcTiTRBJSK-LQLqxE0gRk3uTw?width=1110&height=816&cropmode=none)

-   `리전` 은 아마 수정하지 않아도 기본값으로 사용 가능하며, 버킷 이름을 지정해주고 `다음` 으로 넘어갑니다.

![https://dxq5xg.sn.files.1drv.com/y4mj79fWL6yKebGdZFhWBaobpac7_W9aNFuF3c45QTJl0bZjeKDcKg5cPlEM7AGrar2kKKK7Ssdll2aY3-ORp031jGJRwTk52Jp_LeimeBQyB3s4ttWwNRoRa4Oo5ps8ge487iaHFjYbaGu9yLqSxDClLzCb1USqsGD4X1UmhTKeJcNigv2jc5zhBqCldnnrSEfIL-J8sM_LZe-Cim9GgtX3w?width=1124&height=825&cropmode=none](https://dxq5xg.sn.files.1drv.com/y4mj79fWL6yKebGdZFhWBaobpac7_W9aNFuF3c45QTJl0bZjeKDcKg5cPlEM7AGrar2kKKK7Ssdll2aY3-ORp031jGJRwTk52Jp_LeimeBQyB3s4ttWwNRoRa4Oo5ps8ge487iaHFjYbaGu9yLqSxDClLzCb1USqsGD4X1UmhTKeJcNigv2jc5zhBqCldnnrSEfIL-J8sM_LZe-Cim9GgtX3w?width=1124&height=825&cropmode=none)

-   두번째 화면은 수정할만한 것이 없습니다. 바로 `다음` 으로 갑니다.

![https://dxq8xg.sn.files.1drv.com/y4mBkNaSdo16IDtO4VgUyRie5PIRdx5tMB45HLtFI6FmjdX0JRKN6dmCrOH6_X14Of9TXGJsmv2B3Byds9-yZ1zTbeoemr-ROywxeMuk-vTDfKer1_-gYgWCyDdyEL_1FrHn6Iq3u-ProVAZ3Qd6oihBiMdHzf8lvnluOUcBMa_61-yHyv7xMWSAjvu3a47iZOcVITzUZcojCKxSsYenx84LQ?width=1110&height=817&cropmode=none](https://dxq8xg.sn.files.1drv.com/y4mBkNaSdo16IDtO4VgUyRie5PIRdx5tMB45HLtFI6FmjdX0JRKN6dmCrOH6_X14Of9TXGJsmv2B3Byds9-yZ1zTbeoemr-ROywxeMuk-vTDfKer1_-gYgWCyDdyEL_1FrHn6Iq3u-ProVAZ3Qd6oihBiMdHzf8lvnluOUcBMa_61-yHyv7xMWSAjvu3a47iZOcVITzUZcojCKxSsYenx84LQ?width=1110&height=817&cropmode=none)

-   공개된 소프트웨어로 올리기위해, 퍼블릭 액세스 권한도 풀어줍니다.

![](https://dxq7xg.sn.files.1drv.com/y4mgFlfzuHv2ii0RbYe-x8B0lE_u07hEkkg2dEhmm4YyGN2Jqln6gCyizii2IouSGG5jlXkVFeUjmJwnb7fdlRuaLebyPu3VfLDXc__o2Cl_CyFrqNYquQqbNXmVjAPLCDPSclXhWvP89kczg8sWroR4GjkQg2A4dNlZ-WgDdMNg6T5rCpiAgbykbT5T-xGQRYsFWoz_qAmz9xRyjm5Wod3jw?width=1136&height=791&cropmode=none)

-   설정을 확인하고 `버킷 만들기` 로 완료 합니다.

![https://dxrgxg.sn.files.1drv.com/y4mD7PflCwtcp1nFKYhSWugy-8UxrBL_nRdH_cXyuQsZ43UotrsD7kzxj6Q7R2vQvjsJ0mVmhg55xM7GjKeqV5MGmSos4aeG-Ei94vBCGzc4iD6ThixhmgJHmRXl8BOQ74H9n-YfFtvofYMjYs15kxLut7TVH7yHF1TXgi4vphmZYhlcbUluO7jtX552aLOoossW1c6cCHXqB-4LGnpTbFsUA?width=1629&height=820&cropmode=none](https://dxrgxg.sn.files.1drv.com/y4mD7PflCwtcp1nFKYhSWugy-8UxrBL_nRdH_cXyuQsZ43UotrsD7kzxj6Q7R2vQvjsJ0mVmhg55xM7GjKeqV5MGmSos4aeG-Ei94vBCGzc4iD6ThixhmgJHmRXl8BOQ74H9n-YfFtvofYMjYs15kxLut7TVH7yHF1TXgi4vphmZYhlcbUluO7jtX552aLOoossW1c6cCHXqB-4LGnpTbFsUA?width=1629&height=820&cropmode=none)

-   버킷 설정을 위해 `속성` 에 들어가기

![https://dxrfxg.sn.files.1drv.com/y4mBH3iw0v-1NRH2QQNcuXFK2zEBFmmMisHbyeyb4sqJ7T8poC_6JuXRFubdkZYaZYrCr49ZIhmqUR7B3fjm6lXl6JvvKS-E0WJSINcD7BS9alvdusfZXMKQoHHeJX477xNAsnkH13K0f3Zysjgk3qb1tHfg1MpUBjRIENEgbnVQc_lnp2miPXxx-tGefoNdg0rl0uScIJLNMdKT5zbOjysIA?width=1227&height=839&cropmode=none](https://dxrfxg.sn.files.1drv.com/y4mBH3iw0v-1NRH2QQNcuXFK2zEBFmmMisHbyeyb4sqJ7T8poC_6JuXRFubdkZYaZYrCr49ZIhmqUR7B3fjm6lXl6JvvKS-E0WJSINcD7BS9alvdusfZXMKQoHHeJX477xNAsnkH13K0f3Zysjgk3qb1tHfg1MpUBjRIENEgbnVQc_lnp2miPXxx-tGefoNdg0rl0uScIJLNMdKT5zbOjysIA?width=1227&height=839&cropmode=none)

-   버킷명을 확인하고 `속성` 탭에 들어가면 `정적 웹 사이트 호스팅` 이라는 메뉴에서 정적으로 빌드된 프로젝트의 시작지점(index)를 지정 할 수 있습니다. 또한 상단의 `엔드포인트` 주소를 알고있어야 접속 할 수 있습니다.

![https://fxpvow.sn.files.1drv.com/y4mvzvUn5ZC6WFNaRAUdJVT9td-JVU5HtTmqnMoY5ohPCTvz4OgMxdU_wYuJMH-fhd8-2vvL26lRNDqCIlUlAaNM9qoeK2yV2vrmNDEyMUF0wEJsdwnIslhym77FDbQ7VAXZSwuXOCVEnxsPR0UAclTMEtAwXGaSah5jB7TiX2di771dQofCOecfY4olq96ymy2hz5At5tijI7gRRI3LXhJdw?width=579&height=870&cropmode=none](https://fxpvow.sn.files.1drv.com/y4mvzvUn5ZC6WFNaRAUdJVT9td-JVU5HtTmqnMoY5ohPCTvz4OgMxdU_wYuJMH-fhd8-2vvL26lRNDqCIlUlAaNM9qoeK2yV2vrmNDEyMUF0wEJsdwnIslhym77FDbQ7VAXZSwuXOCVEnxsPR0UAclTMEtAwXGaSah5jB7TiX2di771dQofCOecfY4olq96ymy2hz5At5tijI7gRRI3LXhJdw?width=579&height=870&cropmode=none)

-   정책 수정을 위해 `권한` 탭에 들어가서 `버킷 정책` 에 들어가면 버킷 정책 편집기로 수정이 가능합니다. 결론적으론 하단의 `설명서` 를 참고하여 정책 생성을 완료하면 됩니다. 중간에 버킷 정책 편집기 옆에 `ARN` 을 복사 한 뒤, `정책 생성기` 를 누르고 하단 그림처럼 입력 합니다.

![https://fxpuow.sn.files.1drv.com/y4mZllngyTnvgSfL2q_yvh-fJ7PRYpmb1-0JoZV72l5hg-zghZ-bpGsskWyU0YBf6WncmJ74gdkvGKL44HX73oyCquBtzTTR3Qhf4-iDQRAS1fDx8flWrZ9LOdjqMvq6gmv9dOewfefMNvNXkGk9QzA5J_cyFrc50QE3xOPxlcxT9hDHwCanRv0vLOBs_7d4OeF-A0RVvAr3ZnUbVLtSpGKPw?width=801&height=899&cropmode=none](https://fxpuow.sn.files.1drv.com/y4mZllngyTnvgSfL2q_yvh-fJ7PRYpmb1-0JoZV72l5hg-zghZ-bpGsskWyU0YBf6WncmJ74gdkvGKL44HX73oyCquBtzTTR3Qhf4-iDQRAS1fDx8flWrZ9LOdjqMvq6gmv9dOewfefMNvNXkGk9QzA5J_cyFrc50QE3xOPxlcxT9hDHwCanRv0vLOBs_7d4OeF-A0RVvAr3ZnUbVLtSpGKPw?width=801&height=899&cropmode=none)

-   `Type` 을 `S3` 로 선택하면 하단의 `AWS Service` 는 자동으로 선택되며, 허용을 해야하기에 `Allow`, `*` 전체로 Action을 `GetObject` 로 선택합니다. 또한, 바로 전 화면에서 복사한 ARN 에 끝에 "/\*" 을 붙여서 하단 ARN에 입력 후 정책을 만든 뒤 다시 이전화면으로 넘어가서 정책에 아래처럼 입력 합니다.

![https://fxpwow.sn.files.1drv.com/y4mPqbcj2gcN5DCwNt_L-h6qBDr6I_TKzGU6U46sFwGx3VJLu0z2qQYx0j4NYEKqKnNDZkKybMAkH_OqMHl3qw4tMFFQF4nSC8-4WyNEMJpy6QwXREWQIMVi926h_sqDbvw3qDoQHloXq3RNAmAgtYf7S5bNa1G5PoGLh4ifug5NufSiD5xP02xFGhRCTwZUe305cx-kIUqE4N2PXMLJLb2Bg?width=776&height=777&cropmode=none](https://fxpwow.sn.files.1drv.com/y4mPqbcj2gcN5DCwNt_L-h6qBDr6I_TKzGU6U46sFwGx3VJLu0z2qQYx0j4NYEKqKnNDZkKybMAkH_OqMHl3qw4tMFFQF4nSC8-4WyNEMJpy6QwXREWQIMVi926h_sqDbvw3qDoQHloXq3RNAmAgtYf7S5bNa1G5PoGLh4ifug5NufSiD5xP02xFGhRCTwZUe305cx-kIUqE4N2PXMLJLb2Bg?width=776&height=777&cropmode=none)

-   여기까지 했으면 일단 S3 설정은 완료 했습니다. (테스트삼아 소스를 수동으로 올려보셔도 되긴합니다,,)

## AWS 인증 사용을 위한 AWS IAM 등록하기

![https://dhraxg.sn.files.1drv.com/y4mLSzSZNSPOgyRLCbCjUDUD_D0l2kFPdqunobQcw6CQj5duwOt1JNVft60BhjcIT6HhqMACc_wucoX7sCa--qjNopbtvRc4gUBQ-yQZ-hOtudwlWg-JDOZmO1kvwvzEKshEUKVyDkMW5Jmb-Hs4k_EkVGwFCEla4Bw7pa_iFDQ2hQz1Z4w3QSEC42BGKa5aadJBMBQMzfs4f-mTzP03A2hlw?width=1356&height=548&cropmode=none](https://dhraxg.sn.files.1drv.com/y4mLSzSZNSPOgyRLCbCjUDUD_D0l2kFPdqunobQcw6CQj5duwOt1JNVft60BhjcIT6HhqMACc_wucoX7sCa--qjNopbtvRc4gUBQ-yQZ-hOtudwlWg-JDOZmO1kvwvzEKshEUKVyDkMW5Jmb-Hs4k_EkVGwFCEla4Bw7pa_iFDQ2hQz1Z4w3QSEC42BGKa5aadJBMBQMzfs4f-mTzP03A2hlw?width=1356&height=548&cropmode=none)

-   서비스에서 `IAM` 을 선택하고, `사용자` → `사용자 추가` 를 통해 IAM 사용자를 추가 합니다.

![https://dhqqxg.sn.files.1drv.com/y4mIWWBEWCfw7VgdqU38K2-9bp-4mmv7OwLei7mihBacTTw-0ouzBgraajQktB04DyeMkZwrwPRz_f-2kjUfng8savFicg5H2fTnw2C6cdrPxQt4VmVrjhEdfc33PhFJhe2kHDFjQAhQ4LhsD7qiezIs9OxWVV1f9KjxpQhR-UTVblvEBqehciWO--kuDAX-7i-I2vT1577W4Wwr-UTDoIk-Q?width=963&height=836&cropmode=none](https://dhqqxg.sn.files.1drv.com/y4mIWWBEWCfw7VgdqU38K2-9bp-4mmv7OwLei7mihBacTTw-0ouzBgraajQktB04DyeMkZwrwPRz_f-2kjUfng8savFicg5H2fTnw2C6cdrPxQt4VmVrjhEdfc33PhFJhe2kHDFjQAhQ4LhsD7qiezIs9OxWVV1f9KjxpQhR-UTVblvEBqehciWO--kuDAX-7i-I2vT1577W4Wwr-UTDoIk-Q?width=963&height=836&cropmode=none)

-   `사용자 이름` 과 `API` 및 `CLI` 를 사용하기 위해 액세스 유형에 체크한 뒤 다음으로 넘어갑니다.

![https://dhq6xg.sn.files.1drv.com/y4mM9yWpVaMf4AhNZiHD7FNp8UvkabLabpVGQjQH_OvRlnyWdOIANTKe9tAQXh-i_aLgS0Pwt8nixwLUVnXbv6wgk2TiDNbQ34KQfWhOjEqP8MuO4CtsmIHXtlHjgREWFIgaE74QF5kPsD-c5a7Q6aS7S48XKxYaeTDq5Jp3ZdBO6U0lr564VgUraMgCSa9HvWn0smYp8cn_UgKQw5lnLqkdg?width=976&height=925&cropmode=none](https://dhq6xg.sn.files.1drv.com/y4mM9yWpVaMf4AhNZiHD7FNp8UvkabLabpVGQjQH_OvRlnyWdOIANTKe9tAQXh-i_aLgS0Pwt8nixwLUVnXbv6wgk2TiDNbQ34KQfWhOjEqP8MuO4CtsmIHXtlHjgREWFIgaE74QF5kPsD-c5a7Q6aS7S48XKxYaeTDq5Jp3ZdBO6U0lr564VgUraMgCSa9HvWn0smYp8cn_UgKQw5lnLqkdg?width=976&height=925&cropmode=none)

-   기존 정책에서 `AmazonS3FullAccess` 로 S3 접근권한을 선택하고 다음으로 넘어갑니다. (검색하는게 찾기 편합니다.)

![https://fhpyow.sn.files.1drv.com/y4m4X2HkK8FOVX0Ylda30Q0Bd_kw-W6l3JclY0zAlnGCyTwrATJtZoJ6CiJNKPxDxUwRgfgwH6_ZLB_rHfTYLoNuHrJezesizNx-Ci1AM3-O-FA4G_D3e7rwMLyO31yf2bohblVHbLeEIJJ5bZxi_pNzD4KBxfGy_9GBwp941-aWzyDMN-BP7D7HRcPLz9pfMjTN7QZDeKYACKsm8Bj-UL8xw?width=1006&height=898&cropmode=none](https://fhpyow.sn.files.1drv.com/y4m4X2HkK8FOVX0Ylda30Q0Bd_kw-W6l3JclY0zAlnGCyTwrATJtZoJ6CiJNKPxDxUwRgfgwH6_ZLB_rHfTYLoNuHrJezesizNx-Ci1AM3-O-FA4G_D3e7rwMLyO31yf2bohblVHbLeEIJJ5bZxi_pNzD4KBxfGy_9GBwp941-aWzyDMN-BP7D7HRcPLz9pfMjTN7QZDeKYACKsm8Bj-UL8xw?width=1006&height=898&cropmode=none)

-   뒤로는 특별한 설정이 없어서 화면 3개를 한 이미지에 첨부 했습니다. 다만 `성공` 이후, 나오는 액세스 키와 비밀 액세스 키는 바로 사용해야 하기에 적어두고, 공개하지 않는것이 바람직합니다.

## Github repository에 AWS IAM 연동하기

![https://dhrgxg.sn.files.1drv.com/y4medzou_kvtG5Lw7WP-Q7FcYi1IVWRoNB3sedUN9KBRFPc4FqXlg30bUHCljdDExkelDPxco5xDJOUIZ6KyvlxeHRXtsXjltGwx8oI-uIi1wZGTEF1ZcCtRjvHQv_kuzclEiR1kWNuYTQxvTR080RGqpQsqscrNn27P2WcuBQs_bhtwr4H36IUGUqb-6mGs0O3nB5056pKX9iSG3jA6H5fgA?width=1021&height=887&cropmode=none](https://dhrgxg.sn.files.1drv.com/y4medzou_kvtG5Lw7WP-Q7FcYi1IVWRoNB3sedUN9KBRFPc4FqXlg30bUHCljdDExkelDPxco5xDJOUIZ6KyvlxeHRXtsXjltGwx8oI-uIi1wZGTEF1ZcCtRjvHQv_kuzclEiR1kWNuYTQxvTR080RGqpQsqscrNn27P2WcuBQs_bhtwr4H36IUGUqb-6mGs0O3nB5056pKX9iSG3jA6H5fgA?width=1021&height=887&cropmode=none)

-   기존에 만들었던 repository의 `setting` 으로 들어갑니다.

![https://fxpzow.sn.files.1drv.com/y4mdfsY_BQQjbpQw2z4W3Ltzu1_7wSiV8zoPReW7-cQSKrQUJs0rmtuIxUdDYzz5EdbtetT5zNiURDFWU_rzaSdSAkkYgK1kP9Txo4quvqXWiYoIBDaBQI60ZefZBqRFd96-gkDCx5MzCAwvs627S-CkS4vSvE2g5axgpgPIVqGec2L7UxoP0QH-huKcOepGsX4V1r5AAAPxIMZXa1vN9i5Rw?width=1041&height=696&cropmode=none](https://fxpzow.sn.files.1drv.com/y4mdfsY_BQQjbpQw2z4W3Ltzu1_7wSiV8zoPReW7-cQSKrQUJs0rmtuIxUdDYzz5EdbtetT5zNiURDFWU_rzaSdSAkkYgK1kP9Txo4quvqXWiYoIBDaBQI60ZefZBqRFd96-gkDCx5MzCAwvs627S-CkS4vSvE2g5axgpgPIVqGec2L7UxoP0QH-huKcOepGsX4V1r5AAAPxIMZXa1vN9i5Rw?width=1041&height=696&cropmode=none)

-   Repository의 `Secrets` 에 AWS IAM 값을 설정 할 예정입니다. 이 화면에서 `Add a new secret` 을 선택합니다.

![https://fhpzow.sn.files.1drv.com/y4m7fKHrgJsrsNnD7wQtbOv27IXlC96JjINHClpuP8oKkg3YGQG9utRkWS_yh7sOjPS6okrbp9PhHyRPj5AmCCKv8oaxb8dhjyPX5JkMpBi5rZJN94o5i_CAd2GIg7CH6D4cNy8MAHtwmM4iDSYUzqDSDZaS2I2T-3PFROTqfj40uLkvkseRwoedHMv7semZVuxvc_EpZHX8nhFgVDF0Oybiw?width=999&height=566&cropmode=none](https://fhpzow.sn.files.1drv.com/y4m7fKHrgJsrsNnD7wQtbOv27IXlC96JjINHClpuP8oKkg3YGQG9utRkWS_yh7sOjPS6okrbp9PhHyRPj5AmCCKv8oaxb8dhjyPX5JkMpBi5rZJN94o5i_CAd2GIg7CH6D4cNy8MAHtwmM4iDSYUzqDSDZaS2I2T-3PFROTqfj40uLkvkseRwoedHMv7semZVuxvc_EpZHX8nhFgVDF0Oybiw?width=999&height=566&cropmode=none)

-   AWS IAM 에서 셋팅하고 받아온 `액세스 키ID` 와 `비밀 액세스 키` 를 이미지와 동일하게 설정 합니다.

## Github Action 설정하기

### Github Action 속성으로 이해하기

먼저, `github action` 은 `Workflow` 라는 개념을 사용합니다. `Workflow` 는 말 그대로 "`github repository` 에서 발생 할 수 있는 어떤 작업들(build, test, release ... 등)을 어떤 상황에, 어떠한 순서를, 자동으로 실행 할 지 적어 둔 것" 입니다. 예를 들어, `build workflow` 라고 하면, 제가 "build라는 작업이 어떤 `상황` 이 되면 자동으로 수행되도록 `행동(work)` 과 `순서(flow)` 를 정의 한 것"이며, 이런것 "자동으로 실행 하도록 적어놓은 과정"들을 여러개 정의 할 수 있기에 단어로 `workflows` 라고 부릅니다.

그에 따라, 프로젝트 내에 `.github` 라는 폴더를 만들고, 또 그 안에 `workflows` 폴더를 만들면 자동으로 `github action` 이 사용 가능한 상태로 활성화 되며, 여기에 어떠한 `자동화 과정(workflow)` 을 `.yml` 파일로 작성 해 둔다면 이것을 `github action`이 인식하여 수행하게 됩니다.

어렵기 때문에 예제를 통해 대략적으로 이해해 보겠습니다 (읽어도 이해가 안되면 그 아래의 "맛보기" 챕터의 실행 결과를 먼저보고 이해하셔도 좋습니다). 아래의 예제는 workflow 관련 [blank.yml](https://github.com/bin-e/vue-github-action/blob/master/.github/workflows/blank.yml) 예제입니다.

```
name: CI  --- 1

on: [push]  --- 2

jobs:  --- 3
    build:
    runs-on: ubuntu-latest  --- 4

    steps:  --- 5
    - uses: actions/checkout@v1  --- 6
    - name: Run a one-line script
        run: echo Hello, world!
    - name: Run a multi-line script
        run: |
        echo Add other actions to build,
        echo test, and deploy your project.
```

1.  해당 `workflow` 이름인 `name` 을 정의 합니다. 각각의 workflow들을 구분합니다. [(참조)](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions#name)
2.  `on` 은 "이벤트 트리거"라고 하며, 자바스크립트 등 코드를 작성할 때 만나게 되는 그 "이벤트 트리거" 맞습니다. 즉, "어떤 행위([Event](https://en.wikipedia.org/wiki/Event_(computing)))에 동작 할 것인지 지켜보는 것(Trigger)". 여기서는 `push` 이벤트가 발생했는지 지켜보다가, `repository` 에 `push` 가 발생하면 아래 쓰여진 `flow` 를 수행하는 것 입니다.  
    (특정 이벤트가 없이도 주기마다 수행시키는 `schedule` 트리거도 있긴 합니다. [(참조)](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows#scheduled-events-schedule))
3.  `jobs` 은 `on` 에서 지켜보는 이벤트가 발생하면 수행되는 작업들의 내용입니다. 여기선, `repository` 에 `push` 이벤트가 발생하면, 이벤트 트리거인 `on` 에서 `push` 이벤트 동작을 인식하게 되고, 자동으로 하단의 `jobs` 내부 명령들을 실행기(`GitHub-hosted runner`)가 실행시킵니다. 그래서 `push` 만 발생시킨건데도 지정한 어떤 행동들이 자동으로 발생하도록 합니다.
4.  `build` 는 `jobs` 가 어디서, 어떻게 실행될 지 기재한 내역서인데, 여기서 `runs-on` 이란, 해당 "build 과정" 을 실행하는 실행 환경(공식 문서는 `GitHub-hosted runner` 라고 부릅니다) 입니다. 즉, 여기선 `ubuntu(linux)` 환경의 `latest` 버전에서 아래 동작을 실행 합니다. 다른 실행환경에 대한 내용은 하단의 references나 [이 곳](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners)을 참고 하세요.
5.  `steps` 는 상단의 `jobs` 가 순차적으로 실행할 명령의 단어 그대로 "step" 입니다.
6.  더 깊게 들어가는 내용은 복잡해서 생략하겠지만, `jobs` 가 실행되는 시점에 `github action` 은 가상으로 새로운 `인스턴스` 를 생성하여 해당 `workflow의 jobs` 을 처리합니다. 즉, 이벤트가 발생하여 `on` 에 걸리게되면, 각각의 `jobs` 를 실행하기 위해 가상의 인스턴스들을 생성하고, 그 인스턴스의 `운영체제` 같은 `스펙` 을 `build:` 에 기술하는 것 입니다. 따라서, `build:` 의 사양으로 새로 만들어진 인스턴스에는 빌드나 배포에 사용할 나의 `코드` 가 없기 때문에, `uses` 를 정의하여 내 계정의 소스코드를 가져가서 사용합니다. 가져오는 방법은 Github API에서 정의된 actions을 사용하며 자세한 사항은 하단의 레퍼런스나 [이곳](https://github.com/actions/checkout)을 참고 하세요. 또한, github-action에서 사용할 수 있는 github 환경변수들의 원문은 [여기](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-environment-variables#default-environment-variables) 를 참고하세요.

### 간단한 Github Action 실행시켜 맛보기

-   Github action 생성하기 (맛보기는 UI를 통해 생성하지만, 다른 sample이 뜨거나 IDE를 통한 작성이 더 편한분은 프로젝트에 `.github` 폴더를 만들고 그 안에 `workflows` 폴더를 만든 뒤 이곳에 파일을 만들어서 코드를 작성하면 됩니다.)

![https://ehrgxg.sn.files.1drv.com/y4mfJu6nS36e6BOV7hAWeOagXQQhdkhJk0_B4h8G733fh8ogn-_9LOUWeBQpPbrFOl_1QZ3Zcv3_zc1Y-W88ii1sdoctAkCz3MsVPfV3DbORA7QIpl8n0WjH579GnNsSR0wxGgVYusL6g_NTLyxCeaogfYFBVFDyOra_lnzDyAaOyXpaZDiXr1kis5xqjnSrhVIyZbMMkPkVXOccSXLbaVOmA?width=997&height=606&cropmode=none](https://ehrgxg.sn.files.1drv.com/y4mfJu6nS36e6BOV7hAWeOagXQQhdkhJk0_B4h8G733fh8ogn-_9LOUWeBQpPbrFOl_1QZ3Zcv3_zc1Y-W88ii1sdoctAkCz3MsVPfV3DbORA7QIpl8n0WjH579GnNsSR0wxGgVYusL6g_NTLyxCeaogfYFBVFDyOra_lnzDyAaOyXpaZDiXr1kis5xqjnSrhVIyZbMMkPkVXOccSXLbaVOmA?width=997&height=606&cropmode=none)

-   github repository의 `action` 탭을 찾아 누릅니다.

![https://ehrfxg.sn.files.1drv.com/y4m71Yun_cc4Dci3VRzwLP48DePh7gYeinIPf_7qgdJOhONCRKerfwZbm6J9C6OWzSfLgqYVjTAAKFbuUKUblFihIehNxux9Df9Q17OhyC89VRhhbJjn4MKmjYUplpItXgibz3136SjHfW8p0BjdOev3keZW4I5HnT5-J9g1aP0cWROC6h3mtanRbLt9SPsbA8zDjAAD3mKh4k33wiDoZj1dQ?width=1872&height=727&cropmode=none](https://ehrfxg.sn.files.1drv.com/y4m71Yun_cc4Dci3VRzwLP48DePh7gYeinIPf_7qgdJOhONCRKerfwZbm6J9C6OWzSfLgqYVjTAAKFbuUKUblFihIehNxux9Df9Q17OhyC89VRhhbJjn4MKmjYUplpItXgibz3136SjHfW8p0BjdOev3keZW4I5HnT5-J9g1aP0cWROC6h3mtanRbLt9SPsbA8zDjAAD3mKh4k33wiDoZj1dQ?width=1872&height=727&cropmode=none)

-   미리 정의되어있는 `Simple workflow` 를 선택(Set up this workflow) 합니다.

![https://erqpxg.sn.files.1drv.com/y4mjX_gGVJGqHN6L8ZuCcP0OyS4xvHe209E-8GLMUxv4nP_1FmAOGCqcCGrBaUXJW8WTE29OsGcJ9nn0sL0Oup8TYTJRDXXu1oPdYPqa7wn1K72xB_RUcnpw_GhsS2ZrrPPLIYnGjjC9kTCUhjRKuESz_AIHp_pQQxdoYDyTiIStBk9ScRkIwNuNyJsuVf8aOtp9MCRhzcPVsawG9VobMz_uA?width=1283&height=838&cropmode=none](https://erqpxg.sn.files.1drv.com/y4mjX_gGVJGqHN6L8ZuCcP0OyS4xvHe209E-8GLMUxv4nP_1FmAOGCqcCGrBaUXJW8WTE29OsGcJ9nn0sL0Oup8TYTJRDXXu1oPdYPqa7wn1K72xB_RUcnpw_GhsS2ZrrPPLIYnGjjC9kTCUhjRKuESz_AIHp_pQQxdoYDyTiIStBk9ScRkIwNuNyJsuVf8aOtp9MCRhzcPVsawG9VobMz_uA?width=1283&height=838&cropmode=none)

-   사전 정의되어 있는 그대로 생성 합니다. (그림이 커서 짤렸는데 생성은 오른쪽 상단에 있습니다.)

![https://erq9xg.sn.files.1drv.com/y4mYwpnYaQxJ5Z5KvOcdAMzMBBnPOKbBoMkzcIJfQrhV3CqvPJCd9Q3vqgTG-u9SfU4Foa7hdiPHxbrri1k1ec_qBUdGWeJZ92IP1PinDSzHZ0RzRP68TxtGRNSQPXoJ53b9Itf4A64Ribo90rHfwio89ehYIvJOXOv7ptXQkJAYKnDLa58jiZVt5olZz8z6y1XLtd8dIh9uO7M-rjme1P4CQ?width=1620&height=420&cropmode=none](https://erq9xg.sn.files.1drv.com/y4mYwpnYaQxJ5Z5KvOcdAMzMBBnPOKbBoMkzcIJfQrhV3CqvPJCd9Q3vqgTG-u9SfU4Foa7hdiPHxbrri1k1ec_qBUdGWeJZ92IP1PinDSzHZ0RzRP68TxtGRNSQPXoJ53b9Itf4A64Ribo90rHfwio89ehYIvJOXOv7ptXQkJAYKnDLa58jiZVt5olZz8z6y1XLtd8dIh9uO7M-rjme1P4CQ?width=1620&height=420&cropmode=none)

-   다시 `action` 탭으로 들어가면, 방금 만들었던 `name` 이 `CI` 인 `workflow` 한개가 보일 것 이고, 오른쪽에선 `CI` 로 발생한 `action` 을 한개 (그림에선 2갠데 한개가 보여야 정상) 확인하실 수 있습니다.

### simple workflow action의 동작 이해하기

![https://ebraxg.sn.files.1drv.com/y4myA_e7fyizBslW9A15cnCDUt_S9aVXEgh8DkBMKE2ZjNCpWFyLjDRFebZ4V5kMVGDsKTwLVjKAGSogoKunWJ9l8GVnvZCX3W-7GiewYmPOea7M1zuBNod0KUynRNFHcQEWB0249eEuyoycPHG1z3Tdx_YemaEWDE7zY9EQx1q4bnY5njRuuHgFq7tqnVfl7JHgXSoQLlpbWIB_po505-i2Q?width=1261&height=1062&cropmode=none](https://ebraxg.sn.files.1drv.com/y4myA_e7fyizBslW9A15cnCDUt_S9aVXEgh8DkBMKE2ZjNCpWFyLjDRFebZ4V5kMVGDsKTwLVjKAGSogoKunWJ9l8GVnvZCX3W-7GiewYmPOea7M1zuBNod0KUynRNFHcQEWB0249eEuyoycPHG1z3Tdx_YemaEWDE7zY9EQx1q4bnY5njRuuHgFq7tqnVfl7JHgXSoQLlpbWIB_po505-i2Q?width=1261&height=1062&cropmode=none)

-   한방에 이해가 갈리 없겠지만 몇가지 틀만 눈에 익히고 가면 됩니다.

1.  workflow에서 작성한 `name` 이 결국 `action` 탭의 `name` 이 된다.
2.  `jobs` 는 위에서 설명한 것 처럼 Set up 과정을 거치며, (이때 만들어진 github-host 가상머신의 runner(action을 실행해주는 녀석) 버전이 2.164.0 이다.
3.  `ubuntu-latest` 로 생성했기 때문에 아마 `action` 이 실행된 가상머신의 운영체제는 우분투겠다. 따라서 [이 문서](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners) 를 확인하고 내가 고른 운영체제에 무슨 라이브러리가 기본적으로 깔려있는지 알고있어야 한다.
4.  이것도 역시 위에서 설명한 것 처럼, `actions/checkout` 이라는 코드가 `Syncing repository: bin-e/vue-github-action` 이라는 싱크를 연결 → git 버전을 검사 → git remote add origin 을 상단 싱크 repo로 연결 → remote에서 코드를 받아오는 과정 (스크린샷에선 길어서 생략)
5.  `steps` 에서 정의한 `name` 으로 해당 코드가 `run` 되어 나타나는 것을 볼 수 있다.

-   이렇게 간단하게 샘플코드를 통한 github action을 사용해 봤으니 이제 우리의 프로젝트에 맞는 action을 구현해보도록 하겠습니다.

### Github Repository와 AWS S3 연동하고, Github Action에서 사용하기

상단의 심플코드와 동일한 방법으로 Github 웹 UI를 사용하셔도 되지만, 전 프로젝트 폴더에서 IDE를 사용하여 생성 해 보겠습니다.

![https://fxpbow.sn.files.1drv.com/y4mdB2a4S01x5jQbah2K1aBP0VSxFPPYnzA6R5t5Y5GzvXINxMqbjsh0NeYtuCtEgwSUwqcBjlYdHMvHEsY2b6Ocvq9oYR0FfqbLc2t-LRboGv7brLw8todmathhLZw7R6xJS17isWpe7wSTY2enImG6z-WEM3tWI96tf-R4lYZsbNiMCykPpRmTE09eAcqSsLQFhTOxVneZAQ4k-GF4xKbBw?width=603&height=442&cropmode=none](https://fxpbow.sn.files.1drv.com/y4mdB2a4S01x5jQbah2K1aBP0VSxFPPYnzA6R5t5Y5GzvXINxMqbjsh0NeYtuCtEgwSUwqcBjlYdHMvHEsY2b6Ocvq9oYR0FfqbLc2t-LRboGv7brLw8todmathhLZw7R6xJS17isWpe7wSTY2enImG6z-WEM3tWI96tf-R4lYZsbNiMCykPpRmTE09eAcqSsLQFhTOxVneZAQ4k-GF4xKbBw?width=603&height=442&cropmode=none)

-   VSCode를 사용하여 github action 지정 폴더인 `.github/workflows` 에 새로운 파일을 만들어봅니다. (파일명은 자유) 그리고 해당 파일에 아래 내용을 작성 합니다.

```
name: Main CI # action 명

on: # 이벤트 트리거
  push: # push event에 반응
    branches: # github repository의 branch가 
      - master # master 일 경우만

jobs: # jobs
  build: # GitHub-hosted runners env
    runs-on: ubuntu-18.04 # using Ubuntu 18.04 LTS
    steps: # steps
      - name: git clone # 
        uses: actions/checkout@v2 # 

      - name: npm install # 
        run: npm install # npm install

      - name: build # 
        run: npm run build # code build

      - name: deploy # 배포
        env:
          AWS_ACCESS_KEY_ID: '${{ secrets.AWS_ACCESS_KEY_ID }}'
          AWS_SECRET_ACCESS_KEY: '${{ secrets.AWS_SECRET_ACCESS_KEY }}'
        run: |
          aws s3 cp \
            --recursive \
            --region ap-northeast-2 \
            dist s3://vue-github-action
```

해석하면, 해당 CI 이름은 `Main CI` 로 돌아갈 것이며, `master` 브랜치의 `push` 이벤트에 트리거 되어있습니다. 따라서, `repository` 에 `push` 가 발생하면 `github-host` 는 `Ubuntu-18.04 LTS` 버전을 기반 OS로 한 인스턴스를 생성해서 `Runner (실행기)` 를 돌려서 각각 git clone → npm install → build → deploy 를 실행 할 것입니다. 하나 처음나온 구문은 `deploy` 시에 방금 우리가 저장했던 AWS IAM 키를 사용한 AWS 접근과, `aws cli` 를 사용하여 배포 S3 위치에 `build` 완료한 파일을 `cp(복사)` 하는 명령어가 들어있습니다. (dist는 해당 프로젝트를 build 했을 때 나오는 output 폴더 입니다.)

![https://fxpaow.sn.files.1drv.com/y4mgEP_u-mH5JARSqZrv33gHO_LfuQWIpcmyjx5iElC0HrEAiUR6cDPrJIdLItWcntiQh1T2pjxyVkzhfoeFtF2A5NB9Z1SlqdggnDIzol9quhz-jphULHHzzKQoE-Ijw4HHr6lA1H503e8bEMVDnUopHuUEoVLYPzrfEa0CjOJZ-P2zWLCvNbOb_0MTfsDZMWqzA9Cin6MELAvVcobE2291A?width=874&height=483&cropmode=none](https://fxpaow.sn.files.1drv.com/y4mgEP_u-mH5JARSqZrv33gHO_LfuQWIpcmyjx5iElC0HrEAiUR6cDPrJIdLItWcntiQh1T2pjxyVkzhfoeFtF2A5NB9Z1SlqdggnDIzol9quhz-jphULHHzzKQoE-Ijw4HHr6lA1H503e8bEMVDnUopHuUEoVLYPzrfEa0CjOJZ-P2zWLCvNbOb_0MTfsDZMWqzA9Cin6MELAvVcobE2291A?width=874&height=483&cropmode=none)

아까 심플코드와는 다르게 `npm install` 시에 시간이 미세하게 걸려서, 어디까지 프로세스가 진행되었는지 확인 할 수 있습니다. 또한 완료시 v표시가, 실패시 x표시가 뜨므로 어디를 디버깅해야 할 지 빠르게 수정 할 수 있습니다.

## 결과

**성공적인 github action으로 배포가 모두 v표시가 떴다면, 아까 S3 구축시에 적어놨던 엔드포인트 주소를 접근하여 vue가 잘 동작하는지, 수정 후 push를 날렸을 때 수정이 반영되는지 확인 합니다.**

# \# Discussion

-   이정도로 많은 기능이 있을거라고 생각은 못했는데, 생각보다 push 이외에도 pr, 특정 이슈, 특정 릴리즈에 이벤트 트리거를 설정하여 배포하는 아주 많은 방법들이, 많은 명령어들이, 많은 jobs 액션들이 있습니다.. (이건 나중에 팀프로젝트에서 구축하면서 사용해보고, 다시 글을 써야 겠네요..)
-   여기엔 소개하지 않았지만 github 자체에서 [Marketplace](https://github.com/marketplace?type=actions) 를 통한 Actions 자체를 선택 할 수 있도록 구조화 하고 있습니다. 쉬운예로, 지금당장 github market place의 action을 장바구니에 상품고르듯 골라서 손쉽게 사용할 수 있는 많은 CI/CD가 자동화되어 이미 템플릿 형태로 존재 합니다. (가서 아는 언어나 플랫폼의 action yml 파일을 보면 "와 이렇게도 쓰는구나" 하실 수 있습니다.)
-   ~(이러다가 CICD 자체도 그냥 라이브러리 가져오듯 리포지토리 생성시에 쉽게 가져와서 쓸 수 있도록 github이 아주 완벽한 생태계를 만들어줄지도.. 원한다면 자동으로 만들어줄지도..?)~
-   이렇게 길고 읽기 힘든 글이 될 줄 몰랐는데, 앞으로 이렇게 길어질 것 같은 글은 여러편으로 나눠서 작성해야 할 것 같네요.. 이 글도 기회가 된다면 몇번 더 읽어서 가독성을 좀 올려봐야겠습니다.

# \# REFERENCES

-   [(Official) GitHub Actions Documentation](https://help.github.com/en/actions)
-   (Official) [Automating your workflow with GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions)
-   (Official) [About GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/about-github-actions)
-   (Official) [Workflow syntax for GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)
-   (Official) [Virtual environments for GitHub-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/virtual-environments-for-github-hosted-runners)
-   (Official) [Software installed on GitHub-hosted runners](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/software-installed-on-github-hosted-runners)
-   (Official) [Using environment variables](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-environment-variables)
-   (Official) [Core concepts for GitHub Actions](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/core-concepts-for-github-actions)
-   (Official) [Adding an existing project to GitHub using the command line](https://help.github.com/en/github/importing-your-projects-to-github/adding-an-existing-project-to-github-using-the-command-line)
-   (Wiki) [Event (computing)](https://en.wikipedia.org/wiki/Event_(computing))
-   (Official) [Bucket Policy Examples](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html)
-   (Official) [고급 Amazon S3 기능](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/gsg/S3-gsg-AdvancedAmazonS3Features.html)
-   (Official) [AWS 명령줄 인터페이스](https://aws.amazon.com/ko/cli/)
-   (Official) [User Policy Examples](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-policies-s3.html)
-   (Official) [Step 3: Upload a Sample Application to Your GitHub Repository](https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-github-upload-sample-revision.html)
-   (Official) [developer.github.com/v3/](https://developer.github.com/v3/)
-   (Official) [Checkout V2](https://github.com/actions/checkout)
-   (Official) [Events that trigger workflows](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/events-that-trigger-workflows)

## \# 맺으며

> 누군가에 의해 다시 `쓰여진 것` 은 결국 "하나의 길이 만들어진 것" 과 같습니다. 즉, 이 길은 하나의 "방법론" 일 뿐 "정답"이 아닙니다. "자신에게 꼭 맞는 나만의 정답" 이 필요하시다면, 시작은 이 포스트를 통해서 `이미 만들어져 있는 길` 로 편히 걸어가시되, 글의 끝에선 결국 [만든이의 문서](https://help.github.com/en/actions) 를 참조하여 저보다 더 아름답고 우아한 `내 프로젝트의 길` 을 얻으시길 빕니다..! 감사합니다.

~이렇게 github action이 메인 주제일 줄 알았다면 github-action-vue 로 할껄..~