<a>https://arxiv.org/abs/1603.06155</a>



**발화자의 특성을 반영하는 persona-based 신경망 대화 모델**

# 1. Introduction

* 기존의 neural converstaion model에서는 대답이 일관적이지 않은 문제가 발생

   * ex) Q : 넌 지금 어디 사니? A : 나는 서울에 살아

      ​      Q : 넌 어느 도시에 사니? A : 나는 부산에 살아.

* **실제 대화 처럼 한 사람이 일관적인 대답을 뱉는 모델** 을 만들자

* persona - 발화자의 identity로 구성되어 있다고 생각( 배경 지식,  개인 정보, 대화 방식 등)

* seq2seq 모델 기반

* 두 모델 제시

   1. Speaker Model
   2. Speaker - Adressee Model

*  2개의 dataset(트위터, 티비 대본)에 대해서 실험 - BLEU에서 최대 20%, perplexity에서 최대 12%의 성능 향상 가져옴

# 2. Sequence-to-Sequence Models

* input sequence를 한 타임 스텝에 한 토큰씩 받아서 output sequence를 한 토큰씩 내보냄

- input sequence를 multilayered LSTM을 통하여 고정된 크기의 벡터로 매핑 -> 그 벡터를 output sequence로 decode하기 위하여 다른 deep LSTM 이용

  ​

- input $X = \{x_1, x_2, ..., x_{nX}\} $

  - input gate $i_t$, forgat gate $f_t$, output gate $o_t$
  - $e_t$  - time step t에서 그 단에에 대한 벡터
  - $h_t$ - $e_t$와 $h_{t-1}$이 LSTM 모델을 거쳐서 나온 time step t의 단어 벡터
    - ![Imgur](https://i.imgur.com/BR9dJID.png)

- output $Y = \{y_1, y_2,..., y_{nY}\}$

  - softmax로 다음에 나올 token 예측
    - $f(h_{t-1}, e_{y'})$ - activation function
    - ![Imgur](https://i.imgur.com/wdniUPu.png)
  - *EOS*가 뽑힐 때까지

# 3. Personalized Response Generation
## 3.1. Speaker Model
![Imgur](https://i.imgur.com/MNp7J8T.png)

* speaker(응답자)에 대한 정보를 같이 모델링

   * 각각의 speaker를 벡터로 임베딩
   * 응답 데이터에 기반하여 나이, 주소 등의 특성으로 speaker를 클러스터링

* 각각의 speaker i는 $v_i \in \mathcal{R}^{K\times1} $ 로 나타내어짐

   * ![Imgur](https://i.imgur.com/6CCD6FN.png)

* $v_i$는 speaker i가 포함된 모든 대화에서 공유되며, 역전파를 통해 학습됨

* 장점

   * training set에 특정 speaker에 대한데이터가 없더라도 대답을 추론할 수 있음

   * ex) A와 B는 둘다 영국 영어로 얘기함

      training set에서 A가 '너 어디 사니?' 라는 질문을 받았을 때 '영국에 살아'라고 대답

      -> 이 질문에 대한 B의 데이터가 없더라도 일반화해서 추론 가능
## 3.2. Speaker-Addressee Model
* 질문한 사람이 누구냐에 따라서도 응답이 달라짐

  * ex) 철수 :  안녕

    ​      영희 :  철수야 안녕~

* j가 말하면 i가 대답

  * $v_i$ - speaker i의 임베딩

  * $v_j$ - speaker j의 임베딩

  * $V_{ij}$ - i와 j 사이의 상호작용을 임베딩, $v_i$와 $v_j$의 선형 결합,  $V_{ij} \in \mathcal{R}^{K\times1} $, $W_1, W_2 \in \mathcal{R}^{K\times K} $

    ![Imgur](https://i.imgur.com/DVFkwsh.png)

  * ![Imgur](https://i.imgur.com/hcWAuZv.png)

    ![Imgur](https://i.imgur.com/MJp0ugR.png)

## 3.3. Decoding and Reranking

* Decoding

  * N-best list 생성
  * beam size B = 200
  * 매 스텝마다 EOS 로 끝나는 문장은 N-best list에 추가, 끝나지 않은 B개의 가설에 대해 다시 단어 뽑음

* Reranking

  * 'I don't know' 처럼 계속 똑같은 말만 하는것을 막기 위함

  * N-best list를 다음과 같은 scoring function을 이용하여 rerank

    ![Imgur](https://i.imgur.com/Qc9dzFR.png)

    * $p(R|M, v)$ - 질문 M과 응답자의 id v가 주어졌을 때, 응답 R이 나올 확률
    * $|R|$ - 응답 R의 길이
    *  $p(M|R)$ - inverse seq2seq 모델 학습시켜서 얻음

# 4. Datasets

## 4.1. Twitter Persona Dataset

* 각 유저가 적어도 60번의 3-turn 대화에 포함되어 있음
* development, validation, test set에 모두 한번의 reference
* 4 Layer seq2seq

## 4.2. Twitter Sordoni Dataset

- user 정보를 뺀 기존의 데이터
- (Sordoni et al., 2015)의 데이터 그대로 사용
- 한 메시지 당 10번씩 reference - 앞의 데이터셋과 BLEU 비교 불가

## 4.3. Television Series Transcripts

- Friends, The Big Bang Theory 에서 주연 13명의 대화 수집
- 데이터 양이 적어 Opensubtitles의 데이터로 먼저 seq2seq 모델 학습한 후 이 데이터로 speaker 모델 학습

# 5. Results

## 5.1. Twitter Persona Dataset

![Imgur](https://i.imgur.com/56RTfIL.png)

* MLE 모델에서 성능 향상이 두드러짐

## 5.2. Television Series Transcripts 

![Imgur](https://i.imgur.com/NkuKs3D.png)



## 5.3 Quelitative Analysis

* twitter dataset model에서 대답하는 유저가 다르면 대답도 다른것을 볼 수 있음

![Imgur](https://i.imgur.com/5ugzf5O.png)

* TV transcript dataset의 speaker-address 모델 - 질문하는 사람에 맞춰서 대답함

  ![Imgur](https://i.imgur.com/Q4MwAQQ.png)
