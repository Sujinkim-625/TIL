병렬화(parallelism)란, 여러 개를 동시에 처리하는 기술을 의미하며 머신러닝에서는 주로 여러 개의 디바이스에서 연산을 병렬화하여 속도나 메모리 효율성을 개선하기 위해 사용한다. 병령화는 크게 Data Parallelism, Model Parallelism, Pipeline Parallelism으로 구분된다.

## Data Parallelism
### torch.nn.DataParallel
`torch.nn.DataParallel`은 single-node와 multi-GPU에서 동작하는 multi-thread 모듈이다.

Forward pass
- GPU 0이 master GPU로써 batch를 사용하는 GPU 4개로 나누어준다. (scatter)
- Model을 GPU 수만큼 복제한다.(replicate)
- GPU 0에 올라와있는 모델의 파라미터를 GPU 1,2,3으로 broadcast한다.
- 나누어진 batch를 각각 복제된 model에 forward, Logits을 계산한다.
- forward된 outputs를 master GPU에 다시 모아준다.(gather)
- Logits으로부터 Loss를 계산한다.

![[Pasted image 20250102225701.png]]

Backward pass
* 계산된 Loss를 각 디바이스에 scatter한다.
* 전달받은 Loss를 이용해서 각 디바이스에서 Backward를 수행하여 Gradients를 계산한다.
* 계산된 모든 Gradient를 GPU 0로 Reduce하여 GPU 0에 전부 더한다.
* 더해진 Gradients를 이용하여 Gradient 0에 있는 모델을 업데이트한다.

![[Pasted image 20250102230501.png]]

### torch.nn.DataParallel의 문제점
*single-process multi-thread parallelism이 GIL에 의한 방해를 받는다.*
파이썬은 GIL(Global Interpreter Lock)에 의해 하나의 프로세스에서 동시에 여러 개의 thread가 작동할 수 없다. 따라서 multi-thread가 아닌 multi-process 프로그램으로 만들어서 여러 개의 프로세스를 동시에 실행하게 해야한다. 

GIL이란, 여러 개의 thread가 파이썬 바이트 코드를 한번에 하나만 사용할 수 있게 락을 거는 것을 의미한다. GIL은 파이썬의 메모리 안정성을 보장하기 위해 설계되었는데, 이로 인해 파이썬 multi-thread 프로그램에서 multi-thread가 signle-thread처럼 동작하는 성능 병목 현상을 발견할 수 있다.

GIL의 등장 배경은 다음과 같다.
레퍼런스 카운팅이란 파이썬에서 생성된 객체가 객체를 가리키는 참조의 수를 추적하는 참조 카운트 변수를 가진다는 것을 의미한다.

![[Pasted image 20250102235206.png]]

그림에서 보듯이 우측의 객체는 참조의 수가 2인데, 좌측의 객체는 참조가 없어져 0이 된다. 이 개수가 0에 도달하면 개체가 점유한 메모리가 메모리 가비지 컬렉터에 의해 해제된다. 문제는 이 레퍼런스 카운팅 변수가 멀티 스레드 환경에서 두 스레드가 동시에 값을 늘리거나 줄이는 Race Condition이 발생할 수 있다는 것이다. 이러한 상황이 발생하면 메모리 누수가 발생하거나 객체에 대한 참조가 남아있는데도 메모리를 잘못 해제할 수 있다.
GIL은 그래서 멀티 스레드 프로그램에서 이러한 레퍼런스 카운팅에 의해 발생할 수 있는 문제를 미리 예방하고자 한다.

*하나의 모델에서 업데이트된 모델이 다른 device로 매 스텝마다 복제되어야한다.*
현재의 방식은 각 디바이스에서 계산된 Gradient를 하나의 디바이스로 모아서(Gather) 업데이트를 하는 방식이기 때문에 업데이트된 모델을 매번을 다른 디바이스들로 복제(Broadcast)해야하는데, 이 과정에서 많은 비용이 든다. 
따라서, Gradient를 Gather하지 않고 각 디바이스에서 자체적으로 step()을 수행한다면 모델을 매번 복제하지 않아도 될 것이다.

All-reduce 연산을 통해 각 디바이스에서 계산된 Gradients를 모두 더해서 모든 디바이스에 균일하게 뿌려준다면 각 디바이스에서 자체적으로 step()을 수행할 수 있다.
![[Pasted image 20250102232030.png]]

하지만, All-reduce 연산 또한 매우 비용이 높다.

## Distributed Data Parallelism
Data Parallelism의 문제를 개선하기 위해 등장한 데이터 병렬처리 모델으로, single/multi-node & multi-GPU에서 동작하는 multi-process 모듈이다. 

동작원리는 다음과 같다.
* 각기 다른 GPU를 가진 프로세스에 모델을 복제하고, Rank 0의 프로세스가 다른 프로세스들에게 가중치를 broadcast한다.
* 각 프로세스는 DistributedSampler()를 통해 overlapping 없어 서로 다른 mini-batch data를 load한다.
* 각 프로세스의 GPU에서 Forward pass와 loss를 계산한다.
* backward 시에 각 GPU에서 gradient를 구하고 서로 all-reduce 연산을 한다.

마스터 프로세스를 사용하지 않기 때문에 특정 디바이스로 부하가 쏠리지 않고, 효율적인 방식으로 모든 디바이스의 파라미터를 동시에 업데이트하기 때문에 파라미터를 매번 replicate하지 않아도 된다는 장점을 가지고 있다.

![[Pasted image 20250102233155.png]]

backward()와 all-reduce를 중첩시키는 것이 가장 효율적인 방식이다. all-reduce 네트워크 통신에 해당되며, backward(), step()은 GPU 연산이기 때문에 동시에 처리할 수 있다. 이들을 중첩시키면, computation과 communication이 최대한으로 overlap되기 때문에 연산 효율이 크게 증가한다. backward 연산과 all-reduce 연산을 중첩하는 경우에 backward 연산이 뒤쪽 레이어부터 순차적으로 이루어지기 때문에 계산이 끝난 뒤쪽 레이어부터 먼저 전송하게 된다.

![[Pasted image 20250102234340.png]]

또 all-reduce 연산의 경우 layer마다 이루어지는 것이 아니라 Gradient Bucketing을 수행하여 bucket이 가득찰 때 수행하게 된다. Gradient bucketing은 일정한 사이즈의 bucket에 gradient를 저장해두고, 가득차면 다른 프로세스로 전송하는 방식을 말한다. backward 연산과정에서 뒤쪽부터 계산된 gradient들을 차례로 bucket에 저장하다가 bucket의 용량이 가득차면 all-reduce를 수행해서 각 디바이스에 gradient 평균 값을 계산하여 전달한다. 

 ![[Pasted image 20250102234852.png]]
 
참고자료
https://sonstory.tistory.com/123
https://da2so.tistory.com/21
https://nbviewer.org/github/tunib-ai/large-scale-lm-tutorials/blob/main/notebooks/05_data_parallelism.ipynb
https://bloofer.net/114