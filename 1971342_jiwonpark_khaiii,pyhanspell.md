# khaiii

khaiii는 "Kakao Hangul Analyzer III"의 첫 글자들만 모아 만든 이름으로 카카오에서 개발한 세 번째 형태소분석기입니다. 두 번째 버전의 형태소분석기 이름인 dha2 (Daumkakao Hangul Analyzer 2)를 계승한 이름이기도 합니다.

형태소는 언어학에서 일정한 의미가 있는 가장 작은 말의 단위로 발화체 내에서 따로 떼어낼 수 있는 것을 말합니다. 즉, 더 분석하면 뜻이 없어지는 말의 단위입니다. 형태소분석기는 단어를 보고 형태소 단위로 분리해내는 소프트웨어를 말합니다. 이러한 형태소분석은 자연어 처리의 가장 기초적인 절차로 이후 구문 분석이나 의미 분석으로 나아가기 위해 가장 먼저 이루어져야 하는 과정으로 볼 수 있습니다. (한국어 위키피디아에서 인용)

## 데이터 기반

기존 버전이 사전과 규칙에 기반해 분석을 하는 데 반해 khaiii는 데이터(혹은 기계학습) 기반의 알고리즘을 이용하여 분석을 합니다. 학습에 사용한 코퍼스는 국립국어원에서 배포한 [21세기 세종계획 최종 성과물](https://ithub.korean.go.kr/user/noticeView.do?boardSeq=1&articleSeq=16)을 저희 카카오에서 오류를 수정하고 내용을 일부 추가하기도 한 것입니다.

전처리 과정에서 오류가 발생하는 문장을 제외하고 약 85만 문장, 천만 어절의 코퍼스를 사용하여 학습을 했습니다. 코퍼스와 품사 체계에 대한 자세한 내용은 [코퍼스](https://github.com/kakao/khaiii/wiki/코퍼스) 문서를 참고하시기 바랍니다.

## 알고리즘

기계학습에 사용한 알고리즘은 신경망 알고리즘들 중에서 Convolutional Neural Network(CNN)을 사용하였습니다. 한국어에서 형태소분석은 자연어처리를 위한 가장 기본적인 전처리 과정이므로 속도가 매우 중요한 요소라고 생각합니다. 따라서 자연어처리에 많이 사용하는 Long-Short Term Memory(LSTM)와 같은 Recurrent Neural Network(RNN) 알고리즘은 속도 면에서 활용도가 떨어질 것으로 예상하여 고려 대상에서 제외하였습니다.

CNN 모델에 대한 상세한 내용은 [CNN 모델](https://github.com/kakao/khaiii/wiki/CNN-모델) 문서를 참고하시기 바랍니다.

## 성능

### 정확도

#### v0.3

CNN 모델의 주요 하이퍼 파라미터는 분류하려는 음절의 좌/우 문맥의 크기를 나타내는 win 값과, 음절 임베딩의 차원을 나타내는 emb 값입니다. win 값은 {2, 3, 4, 5, 7, 10}의 값을 가지며, emb 값은 {20, 30, 40, 50, 70, 100, 150, 200, 300, 500}의 값을 가집니다. 따라서 이 두 가지 값의 조합은 6 x 10으로 총 60가지를 실험하였고 아래와 같은 성능을 보였습니다. 성능 지표는 정확률과 재현율의 조화 평균값인 F-Score입니다.

[![img](https://github.com/kakao/khaiii/raw/master/.github/img/win_emb_f.png)](https://github.com/kakao/khaiii/blob/master/.github/img/win_emb_f.png)

win 파라미터의 경우 3 혹은 4에서 가장 좋은 성능을 보이며 그 이상에서는 오히려 성능이 떨어집니다. emb 파라미터의 경우 150까지는 성능도 같이 높아지다가 그 이상에서는 별 차이가 없습니다. 최 상위 5위 중 비교적 작은 모델은 win=3, emb=150으로 F-Score 값은 97.11입니다. 이 모델을 large 모델이라 명명합니다.

#### v0.4

[띄어쓰기 오류에 강건한 모델을 위한 실험](https://github.com/kakao/khaiii/wiki/띄어쓰기-오류에-강건한-모델을-위한-실험)을 통해 모델을 개선하였습니다. v0.4 모델은 띄어쓰기가 잘 되어있지 않은 입력에 대해 보다 좋은 성능을 보이는데 반해 세종 코퍼스에서는 다소 정확도가 떨어집니다. 이러한 점을 보완하기 위해 base 및 large 모델의 파라미터를 아래와 같이 조금 변경했습니다.

- base 모델: win=4, emb=35, F-Score: 94.96
- large 모델: win=4, emb=180, F-Score: 96.71

### 속도

#### v0.3

모델의 크기가 커지면 정확도가 높아지긴 하지만 그만큼 계산량 또한 많아져 속도가 떨어집니다. 그래서 적당한 정확도를 갖는 모델 중에서 크기가 작아 속도가 빠른 모델을 base 모델로 선정하였습니다. F-Score 값이 95 이상이면서 모델의 크기가 작은 모델은 win=3, emb=30이며 F-Score는 95.30입니다.

속도를 비교하기 위해 1만 문장(총 903KB, 문장 평균 91)의 텍스트를 분석해 비교했습니다. base 모델의 경우 약 10.5초, large 모델의 경우 약 78.8초가 걸립니다.

#### v0.4

모델의 크기가 커짐에 따라 아래와 같이 base, large 모델의 속도를 다시 측정했으며 v0.4 버전에서 다소 느려졌습니다.

- base 모델: 10.8 -> 14.4
- large 모델: 87.3 -> 165

## 사용자 사전

신경망 알고리즘은 소위 말하는 블랙박스 알고리즘으로 결과를 유추하는 과정을 사람이 따라가기가 쉽지 않습니다. 그래서 오분석이 발생할 경우 모델의 파라미터를 수정하여 바른 결과를 내도록 하는 것이 매우 어렵습니다. 이를 위해 khaiii에서는 신경망 알고리즘의 앞단에 기분석 사전을 뒷단에 오분석 패치라는 두 가지 사용자 사전 장치를 마련해 두었습니다.

### 기분석 사전

기분석 사전은 단일 어절에 대해 문맥에 상관없이 일괄적인 분석 결과를 갖는 경우에 사용합니다. 예를 들어 아래와 같은 엔트리가 있다면,

| 입력 어절 | 분석 결과    |
| --------- | ------------ |
| 이더리움* | 이더리움/NNP |

문장에서 `이더리움`으로 시작하는 모든 어절은 신경망 알고리즘을 사용하지 않고 `이더리움/NNP`로 동일하게 분석합니다.

세종 코퍼스에서 분석 모호성이 없는 어절들로부터 자동으로 기분석 사전을 추출할 경우 약 8만 개의 엔트리가 생성됩니다. 이를 적용할 경우 약간의 속도 향상도 있어서 base 모델에 적용하면 약 9.2초로 10% 정도 속도 향상이 있었습니다.

기분석 사전의 기술 방법 및 자세한 내용은 [기분석 사전 문서](https://github.com/kakao/khaiii/wiki/기분석-사전)를 참고하시기 바랍니다.

### 오분석 패치

오분석 패치는 여러 어절에 걸쳐서 충분한 문맥과 함께 오분석을 바로잡아야 할 경우에 사용합니다. 예를 들어 아래와 같은 엔트리가 있다면,

| 입력 텍스트 | 오분석 결과                             | 정분석 결과                                |
| ----------- | --------------------------------------- | ------------------------------------------ |
| 이 다른 것  | 이/JKS + _ + 다/VA + 른/MM + _ + 것/NNB | 이/JKS + _ + 다르/VA + ㄴ/ETM + _ + 것/NNB |

만약 khaiii가 위 "오분석 결과"와 같이 오분석을 발생한 경우에 한해 바른 분석 결과인 "정분석 결과"로 수정합니다. 여기서 "_"는 어절 간 경계, 즉 공백을 의미합니다.

오분석 패치의 기술 방법 및 자세한 내용은 [오분석 패치 문서](https://github.com/kakao/khaiii/wiki/오분석-패치)를 참고하시기 바랍니다.

## 빌드 및 설치

khaiii의 빌드 및 설치에 관해서는 [빌드 및 설치 문서](https://github.com/kakao/khaiii/wiki/빌드-및-설치)를 참고하시기 바랍니다.

## Contributing

khaiii에 기여하실 분들은 [CONTRIBUTING](https://github.com/kakao/khaiii/blob/master/CONTRIBUTING.md) 및 [개발자를 위한 가이드](https://github.com/kakao/khaiii/wiki#개발자를-위한-가이드) 문서를 참고하시기 바랍니다.

## Slack

khaiii의 슬랙 주소는 [https://khaiii.slack.com](https://khaiii.slack.com/) 입니다. 슬랙 가입 요청 페이지는 [https://join-khaiii.herokuapp.com](https://join-khaiii.herokuapp.com/) 입니다. 설치 시 발생한 문제에 대해 질문하시거나, 개발에 참여하실 분들은 편하게 가입하셔서 같이 말씀 나누시길 바랍니다.

## License

This software is licensed under the [Apache 2 license](https://github.com/kakao/khaiii/blob/master/LICENSE), quoted below.

Copyright 2018 Kakao Corp. [http://www.kakaocorp.com](http://www.kakaocorp.com/)

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this project except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0.

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.



# 기분석 사전

krikit edited this page on Jul 23, 2020 · [7 revisions](https://github.com/kakao/khaiii/wiki/기분석-사전/_history)

###  Pages 16

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">Home</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/CNN-%EB%AA%A8%EB%8D%B8" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">CNN 모델</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/CNN-%EB%AA%A8%EB%8D%B8-%ED%95%99%EC%8A%B5-%EA%B3%BC%EC%A0%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">CNN 모델 학습 과정</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/Pull-Request-%EB%B0%A9%EB%B2%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">Pull Request 방법</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/To-Do-List" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">To Do List</a></div></summary></details>

- <details class="details-reset" open="" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron js-wiki-sidebar-toc-toggle-chevron-open mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">기분석 사전</a></div></summary><ul class="list-style-none mx-4 px-1" style="box-sizing: border-box; padding-left: var(--base-size-4, 4px)  !important; margin-top: 0px; margin-bottom: 0px; margin-right: var(--base-size-24, 24px)  !important; margin-left: var(--base-size-24, 24px)  !important; padding-right: var(--base-size-4, 4px)  !important; list-style: none !important;"><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%EC%97%94%ED%8A%B8%EB%A6%AC%EC%9D%98-%EC%A2%85%EB%A5%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 엔트리의 종류</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%ED%8C%8C%EC%9D%BC" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 파일</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%ED%8F%AC%EB%A7%B7" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 포맷</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%EB%B9%8C%EB%93%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 빌드</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%EB%A1%9C%EB%94%A9" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 로딩</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%9D%8C%EC%A0%88-%EB%8B%A8%EC%9C%84-%EC%A0%95%EB%A0%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">음절 단위 정렬</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84#%EC%82%AC%EC%A0%84-%EC%9E%85%EB%A0%A5-%EA%B0%80%EC%9D%B4%EB%93%9C%EB%9D%BC%EC%9D%B8" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 입력 가이드라인</a></li></ul></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%8F%84%EC%BB%A4-%EB%B9%8C%EB%93%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">도커 빌드</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%9D%84%EC%96%B4%EC%93%B0%EA%B8%B0-%EC%98%A4%EB%A5%98%EC%97%90-%EA%B0%95%EA%B1%B4%ED%95%9C-%EB%AA%A8%EB%8D%B8%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%8B%A4%ED%97%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">띄어쓰기 오류에 강건한 모델을 위한 실험</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%AC%B8%EC%A2%85-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">문종 프로젝트</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">버전 관리</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-%EB%A8%B8%EC%A7%80-%EB%B0%A9%EB%B2%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">브랜치와 머지 방법</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B9%8C%EB%93%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">빌드 및 설치</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%82%AC%EC%A0%84-%EC%9E%90%EB%8F%99-%EC%B6%94%EC%B6%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">사용자 사전 자동 추출</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%84%A4%EC%B9%98-%EC%9C%84%EC%B9%98%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">설치 위치에 관하여</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">오분석 패치</a></div></summary></details>

- Show 1 more pages…

##### Clone this wiki locally



기분석 사전은 단일 어절에 대해 문맥에 상관없이 일괄적인 분석 결과를 갖는 경우 사용합니다.

## 사전 엔트리의 종류

기분석 사전의 엔트리는 아래와 같은 두 가지 종류가 있습니다.

- 완전 일치: 전체 어절이 완전히 일치하는 경우에 적용되는 엔트리
- 전방 매칭: 어절의 앞부분부터 부분적으로 일치할 경우에도 적용되는 엔트리

다음은 기분석 사전 엔트리의 예시들입니다.

| 번호 | 입력 어절     | 분석 결과                                  |
| ---- | ------------- | ------------------------------------------ |
| 1    | 이더리움*     | 이더리움/NNP                               |
| 2    | 이러쿵저러쿵  | 이러쿵저러쿵/MAG                           |
| 3    | 고통스러워하* | 고통/NNG + 스럽/XSA + 어/EC + 하/VX        |
| 4    | 고통스러웠*   | 고통/NNG + 스럽/XSA + 었/EP                |
| 5    | 고통스러웠다. | 고통/NNG + 스럽/XSA + 었/EP + 다/EF + ./SF |

입력 어절의 끝에 "*"가 있는 1, 3, 4번 엔트리는 전방 매칭 패턴입니다. 가령 1번 엔트리 `이더리움*`의 경우는 "이더리움이", "이더리움을" 등과 같은 여러 어절에 적용됩니다.

반변 2, 5번 엔트리는 완전 일치 어절입니다. 이러한 엔트리들은 입력 어절도 처음부터 끝까지 완전히 일치할 경우에만 적용됩니다. 가령 5번 엔트리 `고통스러웠다.`의 경우 "고통스러웠다"와 같이 마지막 구두점 "."이 하나만 없어도 적용되지 않습니다.

자세히 보면 4번과 5번 엔트리는 서로 포함관계에 있습니다. 가령 "고통스러웠다."라는 어절은 4번과 5번 엔트리 모두에 해당됩니다. 이러한 경우 긴 엔트리를 우선적으로 적용합니다. 그래서 "고통스러웠다."의 모든 7개의 음절들은 기분석 사전의 결과로 채워집니다.

반면에 "고통스러웠다"와 같이 마지막 구두점 "."이 없는 어절은 4번 엔트리에 적용됩니다. 그래서 "고통스러웠"까지 5개의 음절은 기분석 사전의 결과로 채워지고, 마지막 "다" 음절은 기계학습 분류기의 판단에 맡깁니다.

## 사전 파일

`rsc/src` 디렉터리 아래에는 두 개의 파일이 존재합니다.

- preanal.auto: 코퍼스로부터 자동으로 추출한 엔트리가 있는 파일
- preanal.manual: 사용자가 직접 추가할 엔트리를 넣을 파일

필요할 경우 `preanal.my`와 같은 새로운 파일을 추가해도 됩니다. 사전을 빌드하는 프로그램이 `preanal.`로 시작하는 모든 파일을 이용하여 사전을 빌드합니다.

사전에 엔트리를 추가하여 변경된 경우 반드시 `make resource` 명령을 통해 사전을 다시 빌드해 줘야 합니다. 마치 소스 코드를 빌드하여 실행 프로그램을 생성하듯이, `rsc/src` 디렉터리 아래의 사전 소스를 빌드하여 `build/share/khaiii` 아래에 프로그램에서 사용할 수 있도록 구조화된 바이너리 사전(리소스)이 생성됩니다.

## 사전 포맷

기분석 사전의 포맷은 `<어절(패턴)> <탭> <분석 결과>`입니다. 아래는 사전 내용 예시입니다.

```
이더리움*	이더리움/NNP
이러쿵저러쿵	이러쿵저러쿵/MAG
# 아래 엔트리들은 비슷비슷하네요~
고통스러워하*	고통/NNG + 스럽/XSA + 어/EC + 하/VX
고통스러웠*	고통/NNG + 스럽/XSA + 었/EP
고통스러웠다.	고통/NNG + 스럽/XSA + 었/EP + 다/EF + ./SF
```

맨 처음에 "#"으로 시작하는 줄은 무시됩니다. 코드에서 주석과 같습니다만, 가운데에 나타난 "#" 이후로 주석이 되지는 않으므로 반드시 맨 앞에만 사용하시기 바랍니다.

## 사전 빌드

`build` 디렉터리에서 `make resource` 명령으로 전체 리소스를 빌드할 수 있지만, 기분석 사전만 별도로 빌드가 가능합니다. 아래 명령을 통해 기분석 사전만 빌드합니다.

```
cd rsc
mkdir -p ../build/share/khaiii
PYTHONPATH=$(pwd)/lib ./bin/compile_preanal.py --rsc-src=./src --rsc-dir=../build/share/khaiii
```

`--rsc-src` 옵션은 `rsc/src` 디렉터리, 즉 `preanal.auto` 및 `preanal.manual` 파일이 있는 디렉터리입니다. `--rsc-dir` 옵션은 `../build/share/khaiii` 디렉터리로 바이너리 사전을 출력할 곳입니다. 빌드가 성공하면 아래 두 파일이 생성됩니다.

```
preanal.tri
preanal.val
```

## 사전 로딩

기분석 사전에 새로운 엔트리를 추가하여 빌드한 경우 기존에 배포한 경로, 예를 들여 `/usr/local/share/khaiii` 혹은 python site-packages 경로에 전체 사전을 다시 복사해 줘야 합니다. 사전의 설치 경로에 관해서는 [설치 위치에 관하여](https://github.com/kakao/khaiii/wiki/설치-위치에-관하여) 문서를 참고하시기 바랍니다.

아니면, 아래와 같이 사전의 경로를 직접 API의 인자로 전달하여 로딩하여야 합니다.

```
from khaiii import KhaiiiApi

api = KhaiiiApi(rsc_dir='/path/to/custom/khaiii/dictionary')
```

## 음절 단위 정렬

사전을 빌드하는 과정에서 [CNN 모델 문서](https://github.com/kakao/khaiii/blob/master/doc/cnn_model.md)에서 설명한 것처럼 어절과 분석 결과를 정렬하여 어절 내 각 음절에 대한 출력 태그를 생성합니다. 이 정렬 과정에서 오류가 발생할 경우 아래와 같은 에러를 보게 될 것입니다.

```
INFO:root:preanal.manual
INFO:root:preanal.auto
ERROR:root:{M:N} [이더] [리움] []
{M:N} [이/I-NNP 더/I-NNP] [륨/I-NNP] []

ERROR:root:preanal.manual:2: fail to align: "이더리움	이더륨/NNP"
ERROR:root:1 errors
```

위 예제는 어절에 "이더리움"을 넣은 데 반해 분석 결과에 "이더륨"이라고 일부러 넣은 것입니다. 이로써 어절의 "리움" 부분과 분석 결과의 "륨" 부분이 정렬이 되지 않고 남아 결국 정렬에 실패하여 빌드 에러를 발생하게 됩니다. 이러한 경우는 보통 분석 결과 부분의 입력에 실수한 경우가 많으므로 바르게 수정하면 문제없이 빌드되는 경우가 많습니다.

반면에 아래와 같은 경우는 양쪽이 바른 분석 결과임에도 불구하고 정렬에 실패하는 경우입니다.

```
INFO:root:preanal.manual
INFO:root:preanal.auto
ERROR:root:{M:N} [] [해야] [겠습니다.]
{M:N} [] [하/I-VV 여/I-EC 야/I-EC] [하/I-VX 겠/I-EP 습/I-EF 니/I-EF 다/I-EF ./I-SF]

ERROR:root:preanal.auto:79823: fail to align: "해야겠습니다.	하/VV + 여야/EC + 하/VX + 겠/EP + 습니다/EF + ./SF"
ERROR:root:1 errors
```

이것은 어절의 "해야" 부분이 분석 결과 "하/I-VV 여/I-EC 야/I-EC" 부분과 정렬이 이뤄지지 않아서 발생하는 문제입니다. 만약, 해당 부분이 정렬되는 것이 맞다면 `rsc/src/char_align.map` 파일에 아래와 같이 규칙을 추가해 주면 정렬이 되고 빌드가 성공합니다.

```
해야	하/I-VV 여/I-EC 야/I-EC	21
```

포맷은 `<음절들> <탭> <분석 결과 음절과 태그> <탭> <정렬 정보>` 형태입니다. 앞에 두 칼럼은 위에서 설명한 정렬이 되지 않는 두 부분을 뜻하지만 마지막 "21" 부분은 첫 번째 칼럼의 한 음절이 두 번째 칼럼의 몇 음절과 정렬이 이뤄지느냐 하는 정보입니다.

| 음절들 | 분석 결과 음절과 태그 | 정렬 정보 |
| ------ | --------------------- | --------- |
| 해     | 하/I-VV, 여/I-EC      | 2         |
| 야     | 야/I-EC               | 1         |

위에서 보는 것처럼 "해"라는 음절은 분석 결과 음절 2개와 정렬이 되고, "야" 음절은 1개와 정렬이 됩니다. 따라서 세 번째 칼럼의 정렬 정보는 "21"이며, 이는 항상 첫 번째 칼럼의 음절 길이만큼 아라비아 숫자로 표기합니다.

이렇게 정렬이 되고 나면 "해"는 원형 복원이 필요한 "I-VV:I-EC:1"과 같은 복합 태그를 갖게 되므로, 만약 원형복원 사전에 존재하지 않는다면 `rsc/src/restore.dic` 파일에 자동으로 원형복원 정보를 추가합니다. 또한 새로운 복합 태그인 경우 `rsc/src/vocab.out.more` 파일에 자동으로 출력 태그 정보를 추가합니다.

## 사전 입력 가이드라인

CNN 모델의 경우 좌우 문맥을 3개씩 이용하여 음절의 출력 태그를 판단하므로, 길이가 4를 넘어가는 경우 분석 결과를 예측하기 어려운 단점이 있습니다. 기분석 사전은 이를 손쉽게 보완할 수 있는 기능이 아닐까 생각합니다. 따라서 긴 고유명사와 같은 오분석에 대응하기 위해 사용하는 것을 추천드립니다.

기분석 사전은 하나의 어절만을 보고 적용되므로, 좌우에 어떤 어절이 오느냐에 따라 분석 결과가 달라지는 모호성이 있는 경우 사용하면 부작용이 발생하게 되니 주의하시기 바랍니다.

어절과 분석 결과의 음절이 변형되는 경우 정렬이 안 되는 경우가 있는데, `rsc/src/char_align.map`에 무조건 넣으면 이 또한 다른 엔트리들의 정렬에 영향을 미칠 가능성이 있으므로, 음절 단위 정렬에 대한 깊은 이해와 확신을 갖고 신중히 추가하시기 바랍니다.



# 오분석 패치

krikit edited this page on Jul 23, 2020 · [5 revisions](https://github.com/kakao/khaiii/wiki/오분석-패치/_history)

###  Pages 16

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">Home</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/CNN-%EB%AA%A8%EB%8D%B8" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">CNN 모델</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/CNN-%EB%AA%A8%EB%8D%B8-%ED%95%99%EC%8A%B5-%EA%B3%BC%EC%A0%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">CNN 모델 학습 과정</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/Pull-Request-%EB%B0%A9%EB%B2%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">Pull Request 방법</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/To-Do-List" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">To Do List</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">기분석 사전</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%8F%84%EC%BB%A4-%EB%B9%8C%EB%93%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">도커 빌드</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%9D%84%EC%96%B4%EC%93%B0%EA%B8%B0-%EC%98%A4%EB%A5%98%EC%97%90-%EA%B0%95%EA%B1%B4%ED%95%9C-%EB%AA%A8%EB%8D%B8%EC%9D%84-%EC%9C%84%ED%95%9C-%EC%8B%A4%ED%97%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">띄어쓰기 오류에 강건한 모델을 위한 실험</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%AC%B8%EC%A2%85-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">문종 프로젝트</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">버전 관리</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-%EB%A8%B8%EC%A7%80-%EB%B0%A9%EB%B2%95" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">브랜치와 머지 방법</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EB%B9%8C%EB%93%9C-%EB%B0%8F-%EC%84%A4%EC%B9%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">빌드 및 설치</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%82%AC%EC%A0%84-%EC%9E%90%EB%8F%99-%EC%B6%94%EC%B6%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">사용자 사전 자동 추출</a></div></summary></details>

- <details class="details-reset" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%84%A4%EC%B9%98-%EC%9C%84%EC%B9%98%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">설치 위치에 관하여</a></div></summary></details>

- <details class="details-reset" open="" style="box-sizing: border-box; display: block;"><summary style="box-sizing: border-box; display: list-item; cursor: pointer; list-style: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color;"><div class="d-flex flex-items-start" style="box-sizing: border-box; align-items: flex-start !important; display: flex !important;"><div class="p-2 mt-n1 mb-n1 ml-n1 btn btn-octicon js-wiki-sidebar-toc-toggle-chevron-button " style="box-sizing: border-box; position: relative; display: inline-block; padding: var(--base-size-8, 8px)  !important; font-size: 14px; font-weight: var(--base-text-weight-medium, 500); line-height: 1; white-space: nowrap; vertical-align: middle; cursor: pointer; user-select: none; border: 0px; border-radius: 6px; appearance: none; color: var(--color-fg-muted); background: transparent; box-shadow: none; transition: color 80ms cubic-bezier(0.33, 1, 0.68, 1) 0s, background-color, box-shadow, border-color; margin-left: calc(-1*var(--base-size-4, 4px))  !important; margin-top: calc(-1*var(--base-size-4, 4px))  !important; margin-bottom: calc(-1*var(--base-size-4, 4px))  !important;"><svg aria-hidden="true" height="16" viewBox="0 0 16 16" version="1.1" width="16" data-view-component="true" class="octicon octicon-triangle-down js-wiki-sidebar-toc-toggle-chevron js-wiki-sidebar-toc-toggle-chevron-open mr-0"><path d="M4.427 7.427l3.396 3.396a.25.25 0 00.354 0l3.396-3.396A.25.25 0 0011.396 7H4.604a.25.25 0 00-.177.427z"></path></svg></div><a class="flex-1 py-1 text-bold" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-accent-fg); text-decoration: none; flex-grow: 1 !important; flex-shrink: 1 !important; flex-basis: 0%; padding-top: var(--base-size-4, 4px)  !important; padding-bottom: var(--base-size-4, 4px)  !important; font-weight: var(--base-text-weight-semibold, 600)  !important;">오분석 패치</a></div></summary><ul class="list-style-none mx-4 px-1" style="box-sizing: border-box; padding-left: var(--base-size-4, 4px)  !important; margin-top: 0px; margin-bottom: 0px; margin-right: var(--base-size-24, 24px)  !important; margin-left: var(--base-size-24, 24px)  !important; padding-right: var(--base-size-4, 4px)  !important; list-style: none !important;"><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EA%B8%B0%EB%B6%84%EC%84%9D-%EC%82%AC%EC%A0%84-vs-%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">기분석 사전 vs 오분석 패치</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%82%AC%EC%A0%84-%ED%8C%8C%EC%9D%BC" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 파일</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%82%AC%EC%A0%84-%ED%8F%AC%EB%A7%B7" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 포맷</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%82%AC%EC%A0%84-%EB%B9%8C%EB%93%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 빌드</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%82%AC%EC%A0%84-%EB%A1%9C%EB%94%A9" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">사전 로딩</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%9D%8C%EC%A0%88-%EB%8B%A8%EC%9C%84-%EC%A0%95%EB%A0%AC" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">음절 단위 정렬</a></li><li class="my-2" style="box-sizing: border-box; margin-top: var(--base-size-8, 8px)  !important; margin-bottom: var(--base-size-8, 8px)  !important; padding-left: 24px;"><a class="Link--primary" data-analytics-event="{&quot;category&quot;:&quot;Wiki&quot;,&quot;action&quot;:&quot;toc_click&quot;,&quot;label&quot;:null}" href="https://github.com/kakao/khaiii/wiki/%EC%98%A4%EB%B6%84%EC%84%9D-%ED%8C%A8%EC%B9%98#%EC%95%8C%EB%A0%A4%EC%A7%84-%EB%AC%B8%EC%A0%9C" style="box-sizing: border-box; background-color: transparent; color: var(--color-fg-default)  !important; text-decoration: none;">알려진 문제</a></li></ul></details>

- Show 1 more pages…

##### Clone this wiki locally



기계학습 모델에 의해 분석한 결과는 오류가 있을 수 있습니다. 모든 입력에 대해 100% 정확한 형태소 분석기는 현실적으로 불가능합니다. 오분석 패치는 기계학습 모델의 결과로 출력된 오분석을 정분석으로 바로잡기 위한 사용자 사전입니다.

## 기분석 사전 vs 오분석 패치

기분석 사전과 오분석 패치는 아래와 같은 차이점이 있습니다.

| 기분석 사전                  | 오분석 패치                      |
| ---------------------------- | -------------------------------- |
| 기계학습 모델 실행 전에 적용 | 기계학습 모델의 결과에 적용      |
| 분석 속도를 빠르게 함        | 분석 속도가 느려짐               |
| 단일 어절에 한해 적용 가능   | 어절과 형태소 개수에 제한이 없음 |

## 사전 파일

`rsc/src` 디렉터리 아래에는 네 개의 파일이 존재합니다.

- base.errpatch.auto: 코퍼스로부터 base 모델의 오분석 패치를 자동으로 추출한 엔트리가 있는 파일
- base.errpatch.manual: 사용자가 base 모델의 오분석을 수정하기 위해 엔트리를 넣을 파일
- large.errpatch.auto: 코퍼스로부터 large 모델의 오분석 패치를 자동으로 추출한 엔트리가 있는 파일
- large.errpatch.manual: 사용자가 large 모델의 오분석을 수정하기 위해 엔트리를 넣을 파일

오분석 패치는 기계학습 모델의 결과에 적용하는 것이므로, base, large 모델에 대해 각각 한쌍씩 존재합니다. 기분석 사전과 마찬가지로 `base.errpatch.`로 시작하는 모든 파일을 이용하여 빌드합니다. 또한 사전에 엔트리를 추가하여 변경된 경우 반드시 `make resource` 명령을 통해 사전을 다시 빌드해 줘야 합니다.

## 사전 포맷

오분석 패치의 포맷은 한 행에 탭으로 구분된 3개의 열이 있는 형태입니다. 아래는 사전 내용 예시입니다.

| **원문**                                  | **오분석**                               | **정분석**                                    |
| ----------------------------------------- | ---------------------------------------- | --------------------------------------------- |
| 중증급성호흡기증후군                      | 중증급/NNG + 성호흡기/NNG + 증후군/NNG   | 중증/NNG + 급성/NNG + 호흡기/NNG + 증후군/NNG |
| 된다는 것                                 | 되/XSV + ㄴ다/EF + 는/ETM + _ + 것/NNB   | 되/XSV + ㄴ다는/ETM + _ + 것/NNB              |
| 하지만,                                   | \| + 하지/MAJ + 만/EC + ,/SP             | \| + 하지만/MAJ + ,/SP                        |
| # 아래는 설명을 위한 가상의 엔트리입니다. |                                          |                                               |
| 복잡하다.                                 | _ + 복잡/XR + 하/XSV + 다/EC + ./SF + \| | _ + 복잡/XR + 하/XSA + 다/EF + ./SF + \|      |
| 검색질의                                  | \| + 검색/NNG + 질/XSN + 의/JKG + \|     | \| + 검색/NNG + 질의/NNG + \|                 |

원문에 공백을 쓸 수 있으며 이는 어절의 경계를 의미합니다. 만약 원문에 공백이 있다면 오분석 및 정분석에는 그에 대응하는 특수한 형태소인 '_'를 맞춰서 써줘야 합니다. 원문에서 공백을 맨 앞이나 맨 뒤에 넣을 수도 있으므로 지워지지 않게 주의하여 작성하시기 바랍니다.

또 하나 특수한 형태소인 '|'는 입력된 전체의 처음과 끝을 의미합니다. 위 예에서 두 번째 엔트리는 "하지만,"이란 원문이 입력의 맨 앞에 있을 경우에 한해 적용되며, 네 번째 엔트리는 "복잡하다."란 원문이 입력의 맨 뒤에 있을 경우에 한해 적용됩니다. 쉽게 말해 문장의 처음(begin of sentence)과 끝(end of sentence)을 나타내는 것이라 생각하면 됩니다.

마지막 엔트리와 같이 문장의 처음과 끝에 동시에 '|'를 사용하면 문장 내에서 일부가 아니라 정확한 전체 문장에 한해 적용이 가능하므로 부작용을 방지할 수 있습니다. 또한 검색 질의나 짧은 대화문으로 구축한 코퍼스로부터 기계학습 모델이 오분석을 발생하는 경우 자동으로 패치를 생성할 수 있습니다.

## 사전 빌드

`build` 디렉터리에서 `make resource` 명령으로 전체 리소스를 빌드할 수 있지만, 오분석 패치만 별도로 빌드가 가능합니다. 아래 명령을 통해 오분석 패치만 빌드합니다.

```
cd rsc
mkdir -p ../build/share/khaiii
PYTHONPATH=$(pwd)/lib ./bin/compile_errpatch.py --model-size=base --rsc-src ./src --rsc-dir=../build/share/khaiii
```

`--model-size` 옵션은 "base" 혹은 "large" 중 하나로 선택합니다. `--rsc-src` 옵션은 `rsc/src` 디렉터리, 즉 `base.errpatch.auto` 및 `base.errpatch.manual` 파일이 있는 디렉터리입니다. `--rsc-dir` 옵션은 `../build/share/khaiii` 디렉터리로 바이너리 사전을 출력할 곳입니다. 빌드가 성공하면 아래 세 파일이 생성됩니다.

```
errpatch.tri
errpatch.len
errpatch.val
```

## 사전 로딩

기분석 사전과 마찬가지로 오분석 패치에 새로운 엔트리를 추가하여 빌드한 경우 기존에 배포한 경로, 예를 들여 `/usr/local/share/khaiii` 혹은 python site-packages 경로에 전체 사전을 다시 복사해 줘야 합니다. 사전의 설치 경로에 관해서는 [설치 위치에 관하여](https://github.com/kakao/khaiii/wiki/설치-위치에-관하여) 문서를 참고하시기 바랍니다.

아니면, 아래와 같이 사전의 경로를 직접 API의 인자로 전달하여 로딩하여야 합니다.

```
from khaiii import KhaiiiApi

api = KhaiiiApi(rsc_dir='/path/to/custom/khaiii/dictionary')
```

## 음절 단위 정렬

기분석 사전과 마찬가지로 원문의 음절과 분석 결과를 정렬하는 것이 필요합니다. 음절 단위 정렬에 관한 자세한 내용은 [CNN 모델 문서](https://github.com/kakao/khaiii/blob/master/doc/cnn_model.md#음절과-형태소의-정렬)와 [기분석 사전 문서](https://github.com/kakao/khaiii/wiki/기분석-사전#음절-단위-정렬)를 참고하시기 바랍니다.

## 알려진 문제

오분석 패치는 하나의 엔트리에서 두 번의 정렬을 수행합니다. 하나는 원문과 오분석의 정렬이고, 또 하나는 원문과 정분석의 정렬입니다. 원문과 정분석의 정렬에서 오류가 나면 [기분석 사전 문서](https://github.com/kakao/khaiii/wiki/기분석-사전#음절-단위-정렬)에서 설명한 방법대로 `rsc/src/char_align.map` 파일에 규칙을 추가하여 처리하면 됩니다.

그러나 음절 기반 시스템의 구조적인 이유와 속도 향상을 위해 원문과 오분석도 정렬이 필요합니다. 오분석의 정렬을 위해 `rsc/src/char_align.map` 파일에 오분석에 관한 규칙을 추가하면 정분석의 정렬에도 영향을 미치게 되므로 하지 말아야 합니다. 따라서 현재로는 원문과 오분석 사이의 정렬에서 오류가 발생할 경우 엔트리를 추가할 수 없습니다. 이 부분은 해결해야 할 과제로, 추후 업데이트될 때까지 기다려 주시길 부탁드립니다. (이 부분에 대해 기여해 주실 분도 적극 환영합니다.)



KHAiii

카카오의 딥러닝 기반 형태소 분석 오픈소스

기계학습에 사용한 알고리즘은 신경망 알고리즘들 중에서 **Convolutional Neural Network(CNN)을 사용**하였으며, 한국어에서 형태소분석은 자연어처리를 위한 가장 기본적인 전처리 과정이므로 속도가 매우 중요한 요소라고 생각하여, 자연어처리에 많이 사용하는 Long-Short Term Memory(LSTM)와 같은 Recurrent Neural Network(RNN) 알고리즘은 속도 면에서 활용도가 떨어진다고 예상하여 고려 대상에서 제외하였다고 합니다. CNN 모델의 주요 하이퍼 파라미터는 분류하려는 음절의 좌/우 문맥의 크기를 나타내는 win 값과, 음절 임베딩의 차원을 나타내는 emb 값입니다. win 값은 {2, 3, 4, 5, 7, 10}의 값을 가지며, emb 값은 {20, 30, 40, 50, 70, 100, 150, 200, 300, 500}의 값을 가집니다. 따라서 이 두 가지 값의 조합은 6 x 10으로 총 60가지를 실험하였고, 성능 지표는 정확률과 재현율의 조화 평균값인 F-Score라고 합니다.

**CNN : 합성곱 신경망**

**khaiii는 데이터 기반으로 동작하기 때문에 기계학습 알고리즘(딥러닝)을 사용**

입력의 경우 각각의 음절이 분류 대상

문장을 띄어쓰기 단위로 토크나이징(토큰을 형태소단위로 쪼갬)한 뒤, 형태소와 품사를 붙인 형태로 결과 출력

ex)

설치하느라 이틀 걸렸고

설치하느라+이틀+걸렸고

걸렸고 -> 걸리+었+고

**analyzed가 쪼개준 형태소 단위로 튜플을 만들고, 한 문장(혹은 구문)을 한 리스트로 합친다.**

**띄어쓰기에 취약(하지만 v.04에 따르면 띄어쓰기 오류에 대한 여러 모델을 실험하여 개선하였다고 합니다.)** 

 

 

 

 

 

 

 

 

 

오픈소스의 기능과, 이를 선정한 이유 

카카오에서 개발한 한글의 형태소 분석 오픈소스인 khaiii는 문장을 띄어쓰기 단위로 토크나이징(토큰을 형태소단위로 쪼갬)한 뒤, 형태소와 품사를 붙인 형태로 결과 출력해주는 기능을 갖고 있으며, 이는 **analyzed가 쪼개준 형태소 단위로 튜플을 만들고, 한 문장(혹은 구문)을 한 리스트로 합쳐줍니다.** 또한, 신경망 알고리즘은 소위 말하는 블랙박스 알고리즘으로 결과를 유추하는 과정을 사람이 따라가기가 쉽지 않기 때문에 오분석이 발생할 경우 모델의 파라미터를 수정하여 바른 결과를 내도록 하는 것이 매우 어렵습니다. **이를 위해 khaiii에서는 신경망 알고리즘의 앞단에 기분석 사전을 뒷단에 오분석 패치라는 두 가지 사용자 사전 장치를 마련해 두었습니다. 입력의 경우 각각의 음절이 분류 대상입니다.**

**기분석 사전**

기분석 사전은 단일 어절에 대해 문맥에 상관없이 일괄적인 분석 결과를 갖는 경우에 사용합니다. 예를 들어 아래와 같은 엔트리가 있다면,

| 입력 어절 | 분석 결과    |
| --------- | ------------ |
| 이더리움* | 이더리움/NNP |

문장에서 이더리움으로 시작하는 모든 어절은 신경망 알고리즘을 사용하지 않고 이더리움/NNP로 동일하게 분석합니다.

**오분석 패치**

오분석 패치는 여러 어절에 걸쳐서 충분한 문맥과 함께 오분석을 바로잡아야 할 경우에 사용합니다. 예를 들어 아래와 같은 엔트리가 있다면,

| 입력 텍스트 | 오분석 결과                             | 정분석 결과                                |
| ----------- | --------------------------------------- | ------------------------------------------ |
| 이 다른 것  | 이/JKS + _ + 다/VA + 른/MM + _ + 것/NNB | 이/JKS + _ + 다르/VA + ㄴ/ETM + _ + 것/NNB |

만약 khaiii가 위 "오분석 결과"와 같이 오분석을 발생한 경우에 한해 바른 분석 결과인 "정분석 결과"로 수정합니다. 여기서 "_"는 어절 간 경계, 즉 공백을 의미합니다.

**NNB NNP와 같은 단어의 뜻은 형태소 품사의 태그를 의미**

 

 

 

**오픈소스 혹은 프로그램과 상호작용 구상**

가공된 사용자의 검색 정보가 **django**로 온 시점부터 시작: 

가공한 정보가 도착하면 도착정보와 관련된 키워드를 해쉬태그와 매치,**sns**상에서 해쉬태그를 기반으로 추출한 자료들을**MaraiDB**에 전송, **MaraiDB**에서 보내 준 url을 통해 **scrapy**가 크롤링한 내용을 가져오고, 가져온 리스트안의 내용을 파이썬 한글 맞춤법 검사 라이브러리인 **Hanspell**을 이용하여 가공한 뒤에, 가공한 자료에서 **khaiii**를 이용하여 형태소 단위로 쪼개어 **MaraiDB**와 **scrapy**로 전송한 뒤, **MaraiDB**는 데이터베이스에 저장, **scrapy**는 분석한 키워드를 기반으로**place**를 크롤링한 후, 크롤링 한 내용을 **MaraiDB**로 전송 후, **django**에 자료 전송, 추가로 **elasticsearch**에도 쪼개진 키워드 제공가능

처음 구상해봤던 다른 앱과의 상호작용

![img](file:///C:/Users/PC/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

그려본 dfd 초안

![image-20221124022725784](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20221124022725784.png)





**khaiii로 구상해본 기능들** 

---



사전추가기능도있음
형태소를 음절단위로 쪼개서 전달
기계학습기반 알고리즘
형태소를 분석하여  비슷한 형태소조합을 학습시켜  데이터를 쌓아가고 유사어, 동음이의어에 대해 뒤에오는 조사,어미,접미사 를 미리 유추 할 수 있게하여 (파싱)보다 정확한 검색을 도와주는 것을 기대 (동음어는 앞 뒤 내용에 따라 뜻이 천차만별로 바뀌기 때문에 학습이 필요) 

형태소 분석은 자연어 처리의 가장 기초적인 절차로 이후 구문 분석이나 의미 분석으로 나아가기 위해 가장 먼저 이루어져야 하는 과정이라고 볼 수 있다
Konlpy가 gpl라이선스라 충돌을 막기위해 아파치2라이선스인 khaiii사용 

기분석사전,오분석패치 이용하여 정확도개선 

기분석사전
기분석 사전은 단일 어절에 대해 문맥에 상관없이 일괄적인 분석 결과를 갖는 경우 사용합니다. 

사전 엔트리의 종류 

기분석 사전의 엔트리는 아래와 같은 두 가지 종류가 있습니다. 

• 완전 일치: 전체 어절이 완전히 일치하는 경우에 적용되는 엔트리 

• 전방 매칭: 어절의 앞부분부터 부분적으로 일치할 경우에도 적용되는 엔트리 

더 긴 문장을 우선으로 분리


기분석 사전                                  
기계학습 모델 실행 전에 적용 
기계학습 모델의 결과에 적용
분석 속도를 빠르게 함
오분석 패치   
분석 속도가 느려짐
단일 어절에 한해 적용 가능
어절과 형태소 개수에 제한이 없음


데이터 -> 모델 -> 오분석 패치/기분석 사전 

make_vocab(vocab.in/vocab.out 생성) -> vocabulary -> morph, resource->sentence(공백 masked 음절 단위로 쪼개진 문장) -> dataset(음절 to tensor) -> embedder  -> models ->  trainer -> evaluator -> tagger-> char_align -> trie 



오분석패치
오분석된 내용을 정분석하여 추출


아파치2.0 

코드를 수정한 경우, 표시
Apache 로고, 이름 등 상표 사용에 제한을 두고 있음
작업에 Apache 라이선스를 적용하려면 대괄호 "[]"로 묶인 필드를 자신의 식별 정보로 대체하여 다음 상용구 고지를 첨부



형태소 분석을 하는 이유는 주로 형태소 단위로 의미있는 단어 를 가져가고 싶거나 품사 태깅을 통해 형용사나 명사를 추출하고 싶을 때 많이 이용하게 됨 

Khaiii는 python을 사용을 할 수 있게 했지만 내부는 C언어로 돌아감 - 속도 때문에



**시행착오**

---

다른 오픈소스들과 연계에 관해 어긋나는 부분이 보여, 형태소 분석활용의 이유에 대해 조사를 해보니 보통 형태소분석 활용을 많이 하는 부분이 색인어 추출인데 일라스틱서치랑도 연계해서 스크래피가 긁어온 내용을 크롤링해서 긁어오고 검색이들어오면 카이가 쪼갠 단어로 색인어를 추출해서 일라스틱서치에 반환하고 반환받은 색인어를 이용해서 색인을 만든 뒤에 일라스틱서치가 자체 인풋 플러그인을 이용해서 빠르게 검색해서 스크래피로 전달하면 그 내용을 마리아 db를 거쳐 장고가 문서화하는 식으로 dfd설계를 살짝 바꾸는 것이 더 자연스러워 보였습니다. 그리고 스크래피에서 카이를 거치는 부분은 빼고 일라스틱 색인어를 만들어주는 방향으로 가는걸로 하였습니다. 또한, 그 글의 내용과 해시태그를 서로 비교해서 해당 글이 얼마나 신뢰성이 높은가를 체크해주는 것을 기대하여 사람들이 조회수를 올리려고 해쉬태그를 달면 실제 글이랑 해쉬태그랑 비교해서 글의 신뢰성을 판단하고 신뢰성이 낮으면 데이터베이스에 넣지않고 높다면 넣는 과정에서 비교하려면 실제 글을 형태소 분석해서 단어단위로 쪼개줘야하므로 카이가 필요하긴 하다고 생각했었습니다. 마지막으로 이를 장고에서 문서화 해주도록 하였습니다.



이를 바탕으로 그려본 dfd

---

![image-20221125114202544](C:\Users\PC\AppData\Roaming\Typora\typora-user-images\image-20221125114202544.png)

