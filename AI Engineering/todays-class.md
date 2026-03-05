there are four decoding strategies
1. greedy search
2. beam search
3. top-k sampling
4. top=p sampling

1. greedy => 
always pick token with highest probability.

2. beam search =>
=> keep top b tokens at every step
=> choose the sequence with highest probability.
=>

3. top k sampling =>
=> keep top  k tokens and then randomly pick one token..


4. top p sampling =>
Top-K fixes number of tokens.Top-P fixes total probability mass. 
=> keep tokens until cummulative probability >=p

