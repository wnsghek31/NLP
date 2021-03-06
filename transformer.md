## Sequece-to-Sequence model

 

* S2S 모델은 RNN의 가장 발전된 형태(2017/03에는)의 아키텍처. LSTM, GRU 등 RNN cell을 길고 깊게 쌓아서 복잡하고 방대한 시퀀스 데이터를 처리하는데 특화된 모델

  

* 입력된 시퀀스로부터 다른 도메인의 시퀀스를 출력하는 다양한 분야에서 사용 ( 기계번역 , 챗봇 (질문 , 대답) , 내용 요약 , Speech to Text )

 

 

* 인코더와 디코더로 구성되있는데, 인코더는 입력 문장의 모든 단어들을 순차적으로 입력받은 뒤에 마지막에 이 모든 단어 정보들을 압축해서 하나의 벡터로 만드는데 이를 context vector라고 한다. 입력 문장의 정보가 하나의 context vector로 모두 압축되면 인코더는 context vector를 decoder로 전송한다. 디코더는 context vector를 받아서 번역된 단어를 순차적으로 출력

 

* 인코더와 디코더의 내부는 사실 두개의 RNN 아키텍쳐. 입력 문장을 받는 RNN 셀을 인코더 , 출력 문장을 출력하는 RNN 셀을 디코더. LSTM 셀 또는 GRU 셀들로 구성. 

 

 

* seq2seq는 훈련 과정과 테스트 과정의 작동 방식이 조금 다르다.

 

훈련 과정에서는 디코더에게 인코더가 보낸 context vecotr와 실제 정답인 <sos> je suis etudiant 를 입력 받았을때 je suis etudiant <eos> 가 나와야 된다고 정답을 알려주면서 훈련시킨다.

하지만 테스트 과정에서는  디코더는 오직 context vector와 <go>만을 입력으로 받은 후에 다음에 올 단어를 예측고, 그 단어를 다음 시점의 RNN 셀의 입력으로 넣는 행위를 반복 

 

### 테스트 과정

 

![](./Transformer/image/seq2seqtest.png)	

 

* 입력 문장은 단어 토큰화를 통해 단어 단위로 쪼개지고 , 단어 토큰 각각은 RNN 셀의 각 시점의 입력이 된다. **인코더 RNN 셀은 모든 단어를 입력받은 뒤 인코더 RNN 셀의 마지막 시점의 은닉 상태를 디코더 RNN 셀로 넘겨주는데 이를 context vector 라고하는거. 이게 디코더 RNN 셀의 첫번째 은닉 상태로 사용되는것** 

 

* 디코더에서는 초기 입력으로 문장의 시작을 의미하는 <sos>가 들어간다. <sos>가 입력되면, 다음에 등장할 확률이 높은 단어 예측 , 첫 RNN 셀이 je를 예측했다면 je는 두번째 RNN 셀의 입력이 되어 다음에 올 단어인 suis를 예측한고 입력으로 보내고 예측하고를 문장의 끝을 의미하는 <eos> 가 다음단어로 예측 될때까지 반복  

 

 

### 단점

 

 

입력값을 인코더로 고정된 일정 크기의 벡터로 압축시켜서 디코더로 보내는데 , 이 고정된 크기의 벡터로 인해 병목 현상을 발생시켜 번역 품질을 악화 시킬수가있따.

그래서 ~ Attention이 ~ 나옴

 

 

## Attention

 

* attention은 단어의 의미처럼 특정 정보에 좀더 주의를 기울이는 것

 

> model이 수행해야 하는 task가 번역이라면 , source는 영어이고 target은 한국어. "Hi my name is poza"라는 문장과 대응되는 "안녕, 내 이름은 포자야" 라는 문장이 있다. model이 "이름은"이라는 token을 deocde할 때 , source에서 가장 중요한 것은 "name"

 

* 그렇다면, source의 모든 token이 비슷한 중요도를 갖기 보다는 name이 더 큰 중요도를 가지면 된다. 이때 더 큰 중요도를 갖게 만드는 방법이 attention

 

* 기본아이디어는 디코더에서 출력 단어를 예측하는 매 시점(time-step) 마다, 인코더에서의 전체 입력 문장을 다시 한번 참고한다는 점. 단 전체 입력 문장을 전부 다 동일한 비율로 참고하는게 아니라, 해당 시점에서 예측해야할 단어와 연관이 있는 입력 단어 부터 집중해서 본다

 

 

### Attention fuction

 

`Attention(Q,K,V) = Attention value`

 

![](./Transformer/image/attentionfunc.png)

 

어텐션 함수는 주어진 쿼리에 대해서 모든 키와의 유사도를 각각 구함. 그리고 구해낸 유사도를 키와 맵핑되어있는 각각의 값에 반영

그리고 유사도가 반영된 값을 모두 더해서 리턴. 이를 어텐션 값이라고 한다

 

> seq2seq + 어텐션 모델에서

	Q = Query : t-1 시점의 디코더에서의 은닉상태

	K = keys : 모든 시점의 인코더 셀의 은닉 상태들

	V = Values : 모든 시점의 인코더 셀의 은닉상태들

 

### Dot product attention

 

seq2seq에서 사용되는 어텐션 중에서 닷 프로덕트 어텐션과 다른 어텐션의 차이는 단지 중간 수식의 차이. 메커니즘 자체는 동일

닷 프로덕트 어텐션을 이해하고 수식만 바꾸면 다른 어텐션도 이해한것이나 마찬가지

 

![](./Transformer/image/dotproductex.png)

 

위 그림은 디코더의 세번째 셀이서 출력단어를 예측할때 어텐션 메커니즘의 모습

첫번째와 두번째 LSTM 셀은 이미 어텐션 메커니즘을 통해 je suis를 예측했다고 가정한다.

**세번째 LSTM 셀은 출력 단어를 예측하기 위해서 인코더의 모든 입력 단어들의 정보를 다시 한번 참고하고자 한다. 여기서 주목할것은 softmax 함수. softmax에서 나온 값은 I, am, a, student 단어 각각이 출력단어를 예측 할 때 얼마나 도움이 되는지의 정도를 수치화한 값** 위의 빨간 직사각형의 크기로 결과값의 크기를 나타냄. 이렇게 각 입력 단어가 디코더의 예측에 도움이 되는 정도로 수치화 되면 이를 하나의 정보로 담아서 디코더로 전송 (위의 초록색 삼각형). 결과적으로 디코더는 출력 단어를 더 정확하게 예측한다.

 

 

#### 1) attention score 구하기

 

 

인코더에서 각 시점을 1,2 ... t 라고 하고 인코더의 은닉상태를 각각 h1, h2 ... hN라고 하자. (여기 h들을 구할때 Bidirection RNN으로 정방향 h와 역방향 h의 묶음으로 할수도) 디코더의 현재 시점 t에서의 은닉상태를 st라고 한다. 또한 여기서는 인코더의 은닉상태와 디코더의 은닉상태의 차원이 같다고 가정!

기본 RNN에서는 현재 셀의 출력단어를 예측하기 위해서 이전 시점 t-1의 은닉 상태와 이전 시점 t-1의 출력 단어를 쓴다

그런데 어텐션 메커니즘에서는 여기에 추가로 Attention value 라는 값을 필요로 한다. t 번째 단어를 예측하기 위한 어텐션 값을 at라고 정의 하면 

`s_t = F( s_t-1 , y_t-1 , at )`

즉 **attnetion score란 현재 디코더의 시점 t에서 단어를 예측하기 위해, 인코더의 모든 은닉 상태 각각이 디코더의 바로 전 시점의 은닉 상태 s_t-1와 얼마나 유사한지를 판단하는 스코어값**

 

Dot-product attention 에서는 이 스코어 값을 구하기 위해 s_t-1를 전치하고 각각의 은닉상태와 내적을 한다 .

 

![](./Transformer/image/dotproductscore.png)

 

 

#### 2) 소프트맥스 함수를 통한 어텐션 분포 구하기

 

**위의 e^t(Attention score) 에 소프트맥스 함수를 적용하여 모든 값을 합하면 1이 되는 확률 분포를 얻어낸다. 이를 Attention Distribution이라 하며 각각을 Attnetion Weight 라 한다**.

> softmax function을 적용하면 출력값인 I , am , a , student 의 어텐션 가중치는 0.1 , 0.4 , 0.1 , 0.4 같이 되는것

 

#### 3) 각 인코더의 어텐션 가중치와 은닉 상태를 가중합하여 어텐션 값을 구한다

 

어텐션의 최종 결과값을 얻기 위해서 각 인코더의 은닉상태와 어텐션 가중치값들을 곱하고 모두 더하면 Attention value (at) 가된다.

 

![](./Transformer/image/attentionvalue.png)

 

이러한 **어텐션 값 at은 종종 인코더의 문맥을 포함하고 있다고 해서 컨텍스트 벡터 (Context vector) 라고도 불린다. seq2seq에서는 인코더의 마지막 은닉 상태를 컨텍스트 벡터라고 부른다면 어텐션 메커니즘에서는 at를 컨텍스트 벡터라고 부른다**

 

 

#### 4) 어텐션 값과 디코더의 t-1 시점의 은닉 상태를 결합한다.

 

어텐션 함수의 최종값인 어텐션 값 at 를 구했다. 앞서 소개한 방법 s_t = f(s_t-1 ,y_t-1 ,a_t)이 사실은 a_t를 s_t-1와 결합하여 하나의 벡터 v_t로 만드다음 수행하기에 `s_t = f(v_t , y_t-1)` 이 된다.

**이제 기존의 RNN 셀이 다음 단어를 예측하는 메커니즘을 그대로 사용하면 된다. 단지 원래 현재 시점의 은닉 상태를 계산하기 위해사용하던 s_t-1 대신 v_t를 사용하는것 !!**

 

  

### 다양한 종류의 Attention 함수가 있따.

 

 

 

 

 

 

 

##Transformer   ( Attention is all you need )

 

RNN이나 CNN을 사용하지 않고 attention 만으로 만든 모델

(이전엔 S2S 모델에 attention 같이 RNN에 attention을 추가해서)

 

### Long-term dependency problem

 

* sequence data를 처리하기 위해 이전까지 많이 쓰이던 model은 recurrent model이였다. recurrent model은 t번째 output을 만들기 위해 t-1번째 hidden state를 사용하였는데, 이렇게 한다면 자연스럽게 문장의 순차적인 특성이 유지..

* 하지만 recurrent model은 많은 개선에도 long-term dependency에 취약함.

 

> “저는 언어학을 좋아하고, 인공지능중에서도 딥러닝을 배우고 있고 자연어 처리에 관심이 많습니다.”라는 문장을 만드는 게 model의 task 라면 '자연어'를 만드는데 '언어학'이라는 말은 중요하다. 그러나 두 단어 사이의 거리가 가깝지 않아 , '자연어'라는 단어를 만들지 못하고 '딥러닝'을 보고 '이미지'를 만들 수도 있는것.

 

* recurrent model은 순차적인 특성이 유지되는 뛰어난 장점이 있음에도, long-term dependency problem 이라는 단점이 있따

 

* **transformer는 recurrence를 사용하지 않고 attention mechanism 만을 사용해 input과 output의 dependecy를 포착한다**

== encoder에서만 사용하는 recurrent로 context vector를 미리 만들고 연산만해서 만들어내니까 recurrence를 사용하지 않는다는건가? ==

	

 

 

### Parallelization

 

* recurrent model은 학습 시 , t번째 hidden state를 얻기 위해 t-1번째 hidden state가 필요함. 즉 순서대로 계산되야 하기에 병렬 처리가 불가능해 계산 속도 느림

 

* 하지만 **transformer에서는 학습시 encoder에서는 각각의 position에 대해 , 즉 각각의 단어에 대해 attention을 해주기만 하고 decoder에서는 masking 기법을 이용해 병렬 처리가 가능해짐 **

 

 

 

## Model Architecutre

 

 

### Encoder and Decoder structure

 

![](./Transformer/image/tranmodelarchitecture)

 

* encoder는 input sequence (x1 ... xn) 에 대해 continuous representation인 z 으로 바꿔준다.

 

* decoder는 z를 받아 output sequence (y1 ... yn)를 하나씩 만들어 낸다.

 

* 각각의 step에서 다음 symbol을 만들 때 이전에 만들어진 output(symbol)을 이용함.

 

> "저는 사람입니다" 라는 문장에서 "사람입니다" 를 만들 때 "저는" 이라는 symbol을 사용하는 것 이런 특성을 auto-regressive 하다고 한다.

 

* 전체적으로는 stacked-sefl-attention 과 pointwise fc layer들을 사용해서 구함

 

 

### Encoder and Decoder stacks

 

![](./Transformer/image/transencodeco.png)

 

 

### Encoder 

 

* N개의 동일한 layer로 구성.  input $x$가 첫 번째 layer에 들어가게 되고 , layer(x)가 다시 layer에 들어가는 식이다

 

* 각각의 layer는 두개의 sub-layer( multi-head self-attention mechanism 과 position-wise fc layer) 를 가지고 있다.

 

* 모델 전체적으로 각 sub-layer에 residual connection을 사용함. residual connection은 input을 output으로 그대로 전달 하는 것을 말한다. 이때 sub-layer의 output dimension을 embedding dimension과 맞춰준다. x + sublayer(x)를 하기 위해서, 즉 residual connection을 하기 위해서는 두 값의 차원을 맞춰줄 필요가 있따. 그 후에 layer normalization을 적용

 

== residual connection이 residual 값을 더하는건가 ?? ==

 

### Decoder

 

* N개의 동일한 layer로 이루어져 있따

* encoder와 달리 총 3개의 sub-layer로 구성되어 있다. 2개는 기존의 encoder의 sub-layer와 동일하고 multi-head attention을 계산 하는 sub-layer를 추가

* 마찬가지로 sub-layer에 residual connection을 사용 한뒤 layer normalization을 해준다

 

* decoder에서는 encoder와 달리 순차적으로 결과를 만들어내야 하기 떄문에 , self-attention을 변형한다. 바로 masking을 추가 해주는. masking을 통해 position i 보다 이후에 있는 position에 attention을 주지 못하게 한다. 즉 position i에 대한 예측은 앞에 나온것들만 보고하는것.

 

![](./Transformer/image/decoderex.png)

 

* 위의 예시에서 a를 예측할때 a 이후에 있는 b,c에는 attention이 주어지지 않는것. 그리고 b를 예측할때는 b 이전에 있는 a만 attention이 주어질 수 있고 이후에 있는 c는 attention이 주어지지 않는 것

 

 

### Attention

 

이 모델에서 사용한 attention 은 scaled dot product attention 과 multi-head attention

 

#### Scaled Dot-product Attention

 

![](./Transformer/image/scaleddotproductattention.png)

 

* Attention의 input은 d_k dimension의 query 와 key들 , d_v dimension의 value들로 이루어져있따

 

* 이때 모든 query 와 key에 대한 dot-product를 계산하고 루트d_k로 나누어주는데 dot product 하고 루트d_k로 scaling 해주기에 scaled dot-product attention인 것. 그리고 여기에 softmax를 적용하고 value 를 곱한다.

* key와 value는 attention이 이루어지는 위치에 상관없이 같은 값. 이떄 query 와 key에 대한 dot-product를 계산하면 query와 key 사이의 유사도를 구할수 있따. 

* softmax를 거친값을 value에 곱해주면 , query와 유사한 value 일수록, 즉 중요한 value 일수록 더 높은 값을 가지게 됨.

 

 

* **query 와 key , value 에 대해 설명하자면, query가 어떤 단어와 관련되어 있는지 찾기 위해서 모든 key들과 연산한다. 여기서 실제 연산을 보면 query와 key를 dot-product 한뒤 softmax를 취하는데, 의미하는것은 하나의 query가 모든 key들과 연관성을 계산 한뒤 그 값들을 확률 값으로 만들어주는것. 따라서 query가 어떤 key들과 높은 확률로 연관성을 가지는지 알게 되는것**

 

* **key와 value는 사실상 같은 단어를 의미한다. 하지만 두개로 나눈 이유는 key값을 위한 vector와 value를 위한 vector를 따로 만들어서 사용한다. key를 통해서 각 단어와 연관성을 확률로 계산하고, value는 그 확률을 사용해 attention 값을 계산하는 용도** 

 

== 그럼 key와 value는 같은거고 만약 영어,프랑스어 번역이라면 프랑스어의 한단어가 query라면 key들이 영어들의 각 단어들 인가??==

 

 

#### Multi-Head Attention

 

![](./Transformer/image/multiheadattention.png)

 

![](./Transformer/image/multiheadattentionhead.png)

 

 

기존의 attention은 전체 dimension에 대해서 하나의 attention만 적용했다. Multi-head attention 이란 전체 dimension에 대해서 한번 attention을 적용 하는 것이 아니라 전체 dimension을 h로 나눠서 attention을 h번 적용시키는 방법

 

각 query, key , value 의 vecotr 는 linearly 하게 h개로 project 된다. 이후 각각 나눠서 attention 시킨후 h개의 vector를 concat 하면 됨. 마지막으로 vector의 dimension을 d_model 로 맞춰주기위해 matrix 곱하면 끝

 

 

![](./Transformer/image/multiheadattentionexpr.png)

 

== 그래서encoder에서 output dimension 을 512 로 맞춰 줬나0??==

 

 

#### Application of Attention in our Model

 

transformer 에서 multi-head attention을 다음과 같은 방법으로 사용

 

* encoder-decoder attention layer 에서 qeury들은 이전 decoder layer에서 온다. 그리고 encoder 에서 온 key와 value를 사용한다. 따라서 decoder의 모든 위치의 token은 input sequence의 어느 곳이든 attend 할 수 있따.

 

* encoder는 self-attention layer를 가진다. 모든 key,value,query는 같은 sequence에서 온다. 정확히는 이전 layer의 output 에서 온다. 따라서 encoder는 이전 layer의 전체 위치를 attend 할수있따

 

* decoder의 self-attention layer도 이전 layer의 모든 position을 attend 할 수 있는데, 정확히는 자신의 position 이전의 position 까지만 attend 할 수 있다. 직관적으로 이해하면 sequence에서 앞의 정보 만을 참고할 수 있게 한것이다. 이러한 목적을 scaled dot-product를 masking 함으로써 구현했다

 

 

### Position-wise Feed-Forward Networks

 

attention sub-layer에 이어서 fully connected feed-forward network를 거치는데 

이 network는 두개의 linear transformation으로 구성되어 있고 , 두 transformation 사이에 ReLU 함수 사용

![](./Transformer/image/positionffn.png)

 

W1X+b1 하고 ReLU (=max(0,x)) 취한다음 W2X+b2 한거구만

 

linear transformation == 선형 변환

 

 

 

 

### ==Embeddings and Softmax==

 

 

 

* embedding 값을 고정시키지 않고, 학습을 하면서 embedding 값이 변경되는 learned embedding을 사용했다. 이떄 input과 output은 같은 embedding layer를 사용함

 

* 또한 decoder output을 다음 token의 확률로 바꾸기 위해 learned linear transformation과 softmax 를 사용. learned linear transformation을 사용했다는 것은 decoder output에 weight matrix W를 곱해주는데 이떄 W가 학습 된다는것

 

 

### Position Encoding

 

recurrence나 convolution을 사용하지 않았기 떄문에 , 추가적으로 위치 정보를 넣어줘야 한다.

따라서 "positional encoding"을 사용해서 input embedding에 위치 정보를 넣어준다

각 위치에 대해서 embedding과 동일한 dimension을 가지도록 encoding을 해준 뒤 그 값을 embedding값과 더해서 사용한다

 

position encoding에 여러 방법이 있찌만 sin, cos 함수로 사용해서 구현 가능 

각 위치 pos와 dimension i에 대한 positional encoding 값은 다음과 같다

 

 

![](./Transformer/image/positionencexpr.png)

 

 

### Why self-attention

 

이 모델에서 recurrentce 나 convolution을 사용하지 않고 self-attention 만을 사용한 이유는 

 

* 레이어당 전체 연산량이 줄어든다

* 병렬화가 가능한 연산이 늘어난다

* long-range의 term들의 dependency도 잘 학습할 수 있게 된다

 

* attention 사용하면 모델의 동작을 해석하기 쉬워짐, attention 하나 뿐 아니라 multi-head의 동작 또한 이해하기 쉬워짐

 

attention의 메커니즘

![](./Transformer/image/attentionmechanism.png)

 

 

 

### Training

 

논문에서 traning에 쓰인것은

 

#### Training data and batching

 

* WMT 2014 English-German dataset의 450만 개의 영어-독일어 문장 쌍 과 WMT 2014 English-French dataset의 360만 개의 문장 쌍 

* 학습시 대략 25000개의 token을 포함하는 문장 쌍을 하나의 배치로 사용

 

#### Optimizer

 

* Adam Optimizer 사용 . 하이퍼 파라미터로는 B1 = 0.9 , B2 = 0.98 , e(엡실론) = 10^-9 

* 학습률의 경우 학습 경과에 따라 변화하게 만들었따. 아래의 공식으로

![](./Transformer/image/lrexpr.png)

	warmup_step의 값으로는 4000을 사용

 

#### Regularization

 

* Residual dropout

* Attention dropout

* label smoothing 

 

이 세개를 정규화를 위해 썼는데, dropout 값  label smoothing 값은 0.1로 설정

 

### Conclusion

 

번역에서 Transformer는 다른 모델들 보다 훨씬 빠르게 학습했고 성능도 더 좋다

이 모델이 크게 의미하는 바는 빠른 학습 속도를 보여준것. 따라서 큰 input을 가지는 문제에 적용할수있다

 

 

 

 

참조

https://pozalabs.github.io/transformer/

https://reniew.github.io/43/  (좋음)