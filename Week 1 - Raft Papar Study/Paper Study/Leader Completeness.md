--- 
- 한 번 커밋된 엔트리는 다음 임기의 리더들에게 반드시 포함될 것을 보장
- 다시 한번 [[AppendEntries.md|append entries]]의 동작을 되짚어보겠습니다.

리더는 새로운 엔트리를 다른 서버로 전파할 때 `AppendEntries` RPC를 호출합니다.  
문제 없이 성공한다면 로그엔트리를 복사하고, 실패한다면 인덱스 값을 1씩 감소시켜서 성공할 때까지 반복하고 성공한다면 로그를 복사합니다.

그것을 과반수의 follower들에게 전파를 성공한다면 위에 언급한 것처럼 commit을 하게 됩니다.

여기서 살펴봐야 할 것은 두가지 입니다.

1. **Election Restriction** (선거 제약)
    - Candidate는 Leader가 되려면 **과반수** 노드의 투표가 있어야 함
    - `RequestVote` [[RequestVote.md|RPC]]에는 Candidate의 마지막 로그의 index와 [[Week 1 - Raft Papar Study/Paper Study/term.md|term]]이 파라미터로 포함되어있는데, 요청을 받은 Follower의 index나 term이 더 높으면 요청을 거절함
2. **Commit Rules** (커밋 규칙)
    - Raft에서는 반드시 **현재 임기의 로그**가 복제되어야만 커밋으로 간주한다.

Raft에서는 위 두가지 규칙을 통해서 Leader Completeness를 보장하고 있습니다.

![[Pasted image 20240720040928.png]]

한 번 순서를 따라가보도록 하겠습니다.

(a) : 2대리더는 1번노드입니다. 2번 index까지 과반수의 노드에 전파가 되어서 커밋이 되었습니다. (`commitIndex`=2)  
(b) : 3번 index가 다수의 노드에 전파되어 커밋되기전에 -> 5번노드가 각 노드에게 투표를 요청합니다. -> 1번과 2번노드에게는 **선거제약** 규칙때문에 투표를 거절당하고, 3번,4번노드에게 표를 받아서 5번노드가 3대리더가 되었습니다. (`commitIndex`=2)  
(c) : 다시 1번노드가 2~4번 노드에게 표를 받아 4대리더가 됩니다. -> 그리고 3번 index를 노드3번에 복사하여 과반수 노드에 전파를 성공했습니다. -> 하지만 이는 **커밋규칙**에 따라 현재임기의 로그가 아니므로 커밋되었다고 할 수 없습니다.  
(d) : 만약 5번노드가 2~4번 노드에게 표를 받아 다시 리더가 된다면 이전 것을 덮어씌우기 때문입니다. (`commitIndex`=3)  
(e) : 하지만 (c)상황에서 1번노드가 4번 index까지 무사히 과반수 노드에 전파를 성공해 커밋한다면 5번노드는 당선될 수 없습니다.(`commitIndex`=4)

일련의 과정을 통해 리더가 바뀌더라도 이전에 커밋했던 index들은 계속 유지된다는 것을 확인할 수 있습니다.

- 커밋은 과반수 노드에 전파되어야 함
- 리더는 과반수 노드의 투표가 있어야 당선됨