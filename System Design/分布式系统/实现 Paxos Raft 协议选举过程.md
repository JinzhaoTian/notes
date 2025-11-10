Paxos 和 Raft 都是用于解决分布式系统中一致性问题的算法，它们确保在多个节点之间即使出现故障也能达成一致的状态。

## Raft 选举过程实现

[Raft](../../../Distributed%20System/Raft.md) 的选举过程相对直观，以下是关键实现步骤：

### 1. 节点状态

```python
class RaftNode:
    def __init__(self):
        self.state = "follower"  # 或 "candidate", "leader"
        self.current_term = 0
        self.voted_for = None
        self.election_timeout = random_timeout()
```

### 2. 选举触发

当 follower 在选举超时内未收到 leader 的心跳：

```python
def start_election(node):
    node.state = "candidate"
    node.current_term += 1
    node.voted_for = node.id
    votes_received = 1
    
    # 向其他节点发送请求投票RPC
    for peer in peers:
        send_request_vote(peer, node.current_term, node.id, 
                        last_log_index, last_log_term)
```

### 3. 处理投票请求

```python
def handle_request_vote(term, candidate_id, last_log_index, last_log_term):
    if term < self.current_term:
        return False  # 拒绝
    
    if (self.voted_for is None or self.voted_for == candidate_id) and 
       is_candidate_log_up_to_date(last_log_index, last_log_term):
        self.voted_for = candidate_id
        reset_election_timeout()
        return True
    return False
```

### 4. 成为leader

当候选者获得多数票：
```python
def become_leader():
    self.state = "leader"
    for peer in peers:
        next_index[peer] = len(log)
        match_index[peer] = 0
    start_sending_heartbeats()
```

## Paxos 选举过程实现

[Paxos](../../../Distributed%20System/Paxos.md) 的选举（Leader 选举）通常通过 Multi-Paxos 变体实现，基本 Paxos 本身没有显式选举：

### 1. 准备阶段(相当于选举)

```python
def prepare(proposer_id, proposal_number):
    promises = []
    for acceptor in acceptors:
        response = acceptor.receive_prepare(proposal_number)
        if response.promised:
            promises.append(response)
    
    if len(promises) >= majority():
        # 进入接受阶段
        accepted_value = choose_value(promises)
        accept(proposal_number, accepted_value)
```

### 2. 接受者处理准备请求

```python
def receive_prepare(n):
    if n > self.promised_number:
        self.promised_number = n
        return PrepareResponse(
            promised=True,
            accepted_number=self.accepted_number,
            accepted_value=self.accepted_value)
    return PrepareResponse(promised=False)
```

### 3. 接受阶段

```python
def accept(proposal_number, value):
    accept_responses = []
    for acceptor in acceptors:
        response = acceptor.receive_accept(proposal_number, value)
        accept_responses.append(response)
    
    if len([r for r in accept_responses if r.accepted]) >= majority():
        # 值被选定
        learn_value(value)
```

