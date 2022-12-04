Kospeech
=============
종단 간 음성 인식 모델을 개발하기 위해 PyTorch에 구축된 Apache 2.0 ASR 연구 라이브러리

<hr>

 한국어 자동 음성 인식 기능<br>

오픈 소스 소프트웨어인 KoSpeech 는 딥 러닝 라이브러리인 PyTorch를 기반으로 하는 모듈식 확장형 한국어 자동 음성 인식(ASR) 툴킷입니다. 여러 자동 음성 인식 오픈 소스 툴킷이 출시되었지만 모두 영어와 같은 비한국어를 처리합니다(예: ESPnet, Espresso). 
<br>
<br>
AI Hub는 KsponSpeech로 알려진 한국어 음성 코퍼스 1,000시간을 공개했지만 모델 성능을 비교할 수 있는 전처리 방법과 기준 모델이 확립되어 있지 않습니다. <br>
<br>
따라서 KsponSpeech 말뭉치에 대한 전처리 방법과 여러 모델(Deep Speech 2, LAS, Transformer, Jasper, Conformer)을 제안합니다.<br>
<br><br>


End to End 모델을 사용
-----------
한국어의 STT 난이도가 높은 이유는 많은 자음과 모음의 조합뿐 아니라 음소, 음절 등 고려해야 하는 사항들이 너무 많다는 것입니다. <br>
그런데, 2012년 이후 딥러닝이 발전하면서 고려해야 하는 사항들이나 다양한 변수들에 대한 처리를 layer에 맡기는 것이 가능해졌습다. <br>
따라서 음소나 음절의 처리, 문법, 발음 등을 모두 학습하게 하는 End to End 모델이 가능해졌으며 해당 서비스에 적용할 kospeech 또한 End to End 모델을 사용하여 처리합니다. <br>
요약하면, End to End 모델은 음성 데이터가 포함하는 문법, 발음 등 여러 특징을 모두 모델이 학습하도록 하여 input으로 Raw Audio를 통째로 넣을 수 있도록 한 것입니다. <br><br><br>

전처리 지원
---------

STT를 구현하기 위해서 음성 데이터와, 이를 전사를 한 Label이 필요합니다. <br>
전사는 말소리를 음성 문자로옮겨 적는 것이다. 더 자세하게 어, 그, 음 와 같은 간투어, 주변 소음 등에 따라 녹음된 내용을 사람이 기록해 놓은 것을 말합니다.<br>
사용자는 데이터의 특징에 따라 전사규칙을 삭제하고 문장 부호를 삭제하는 등의 전처리를 진행해야 했으나 kospeech는 개발자들이 사용한 ai-hub의 데이터(kspon)와 libri에 한해 전처리 과정을 공개했습니다.<br>
<br><br><br>


파이프 라인
------------
Raw audio을 입력하면 학습된 feature transform을 거치도록 합니다. <br>
Acoustic model에 추출한 특징들이 전달되고 해당 모델이 이 특징을 통해 발화 예상을 가능하게 한다. 특징들이 모델과 CTC 알고리즘 통과하며 텍스트로 출력됩니다.<br>
CTC(Connectionist Temporal Classification)는 신호와 텍스트 사이의 alignment를 알기 어려움을 해소하기 위해 적용하는 알고리즘입니다. CTC기반의 음성인식은 초기에 제안된 간단한 형태로 학습 과정에서 음성이 인코더를 통과한 결과에 소프트맥스(Softmax)를 취하고 여기에서 CTC 손실을 계산한 뒤 이를 줄이도록 학습합니다.
<br><br><br>


모듈 설치
-----------
```!pip install -r requirements_cssiri.txt```

<br>
- Python 3.8을 사용했습니다.<br>
- kospeech가 제공하는 다양한 Acoustic Model 중, ds2(deepspeech2)를 사용했습니다.<br>
- Pytorch의 경우 1.10 버전이 사용되기 때문에 상위 버전을 사용하시는 경우 별도로 Pytorch를 재설치해주어야 합니다.<br>
- 전처리, 학습, 예측, 예측한 결과 저장에 필요한 모든 모듈을 포함시켰습니다.<br><br><br><br>

전처리(Preprocess)
------------
```!python ./dataset/kspon/main.py --dataset_path $dataset_path --vocab_dest $vacab_dict_destination ---- -- output_unit 'character' --preprocess_mode 'phonetic' ```
<br>
- output_unit과 preprocess_mode는 상황에 맞게 지정해주시면 됩니다.<br>
- ./dataset/kspon/preprocess/preprocess.py의 line 95~101을 확인해보시면, './'의 위치에 'train.txt' 파일을 필요로 합니다. 해당 파일은 '음성 파일 경로' + '\t' + '한국어 전사' 의 형식으로 작성되어야 합니다.<br>
train.txt를 만들 때 사용한 코드는 ./etc/traintext 생성.ipynb 에 올려져 있습니다.<br><br><br><br>

학습(Train)
-------------------
```!python ./bin/main.py model=ds2 train=ds2_train train.dataset_path=$dataset_path```
<br>
- 학습과 관련된 configs(epoch, batch_size, spec_augment, 음성 파일 확장자 등)의 수정은 ./configs/audio/fbank.yaml 혹은 ./configs/train/ds2_train.yaml 에서 하실 수 있습니다.<br><br><br><br>

kospeech 라이선스
--------------
- Apache License 2.0<br>
GPL과는 달리 소스 코드 공개의 의무가 없고, 2차 라이선스와 변형물의 특허 출원이 가능합니다. <br>
아파치 라이선스 (2.0 기준)은 누구나 해당 소프트웨어에서 파생된 프로그램을 제작할 수 있으며 저작권을 양도, 전송할 수 있는 라이선스 규정을 의미합니다. 
아파치 라이선스에 따르면 누구든 자유롭게 아파치 소프트웨어를 다운 받아 부분 혹은 전체를 개인적 혹은 상업적 목적으로 이용할 수 있으므로 충돌이나 문제가 발생하지 않을 것으로 예상다.<br>
<br><br><br>
References <br>
Wer, Cer 관련: https://holianh.github.io/portfolio/Cach-tinh-WER/ <br>
kospeech: https://github.com/sooftware/kospeech


<br><br><br><br><br><br><br><br><br>

elasticsearch
==========

엘라스틱 서치는 검색 엔진입니다. 여기서 검색 엔진(search engine)이란, 웹에서 정보를 수집하여 검색 결과를 제공하는 프로그램입니다. <br>
<hr>

Elasticsearch는 Elastic Stack 의 핵심인 분산형 RESTful 검색 및 분석 엔진입니다. <br>
Elasticsearch를 사용하여 다음에 대한 데이터를 저장, 검색 및 관리할 수 있습니다. <br>

- 로그
- 측정항목
- 검색 백엔드
- 애플리케이션 모니터링
- 엔드포인트 보안

엘라스틱서치에서는 비정형 데이터를 색인하고 검색하는 것이 가능하며 역색인 구조을 사용함으로써 빠른 검색이 가능합니다.<br>

Elasticsearch를 설정하는 가장 간단한 방법은 Elastic Cloud에서 Elasticsearch Service 로 관리형 배포를 생성하는 것 입니다.<br>

Elasticsearch를 직접 설치하고 관리하려면 https://www.elastic.co/kr/downloads/elasticsearch 에서 최신 버전을 다운로드할 수 있습니다.<br>

<br><br><br>

데이터 추가
--------------
REST API를 통해 JSON 개체(문서)를 전송하여 데이터를 Elasticsearch로 인덱싱합니다. <br>
구조화된 텍스트, 구조화되지 않은 텍스트, 숫자 데이터 또는 지리 공간 데이터가 있든 관계없이 Elasticsearch는 빠른 검색을 지원하는 방식으로 이를 효율적으로 저장하고 인덱싱합니다.
<br><br>

오픈소스 검색엔진
--------
엘라스틱 서치는 아파치 재단의 루씬을 기반으로 개발된 오픈소스 검색엔진입니다.<br><br>

전문 검색
----------
전문 검색이란, 내용 전체를 색인해서 특정 단어가 포함된 문서를 검색하는 것이다. 엘라스틱서치는 이러한 전문 검색이 가능합니다.<br><br>

통계분석
----------
비정형 로그 데이터를 수집하고 한곳에 모아 통계 분석을 할 수 있습니다.<br><br>

역색인
------------
역색인 구조를 통해 특정 단어를 찾을 때 문서 전체에서 찾는 것이 아니라 단어가 포함된 특정 문서의 위치를 알아내어 빠르게 결과를 찾아낼 수 있습니다.<br><br>



라이선스
------------

- Elastic License 2.0 <br>
Elastic 2.0는 오픈소스 라이선스의 거의 모든 자유를 허용하며 소프트웨어 수신자는 소프트웨어를 자유롭게 사용, 변경 및 재배포 할 수 있습니다.






