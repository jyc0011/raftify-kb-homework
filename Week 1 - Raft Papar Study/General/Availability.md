--- 
###### 가용성
- 하나 이상의 노드가 작동을 멈춰도 응답을 받을 수 있음
- 즉, 시스템이 얼마나 응답이 가능하느냐
- Master 노드가 작동X -> 투표를 통해 Slave 노드 중 Master를 선출하고 승격하는 과정 필요
- Slave 노드가 작동을 멈출 경우, 여분의 Slave 노드와 Master 노드가 그 역할을 대신