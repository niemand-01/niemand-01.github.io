---
title: "Deep Q-learning"
excerpt_separator: "<!--more-->"
categories:
  - AI
tags:
  - Deep Learning 
  - Reinforcement Learning 
  - Trading Algo
---

Recently, some articles appear in my linkedin thread which describe an approach of applyign RL in trading algorithm.
I have learned a lot about DL, traditional ML and CV stuff during my master, but still RL is a new continent for me.

Deep Q learning is a combination of Deep Learning and Q-learning, where Q-learning is a subcategory of Reinforcement learning 
which is named after its action-value function Q.

For readers who have interest on digging into mathematical source of Q-learning, i recommand reading these zhihu tutorials: [DQN3](https://zhuanlan.zhihu.com/p/21340755?refer=intelligentunit),[DQN4](https://zhuanlan.zhihu.com/p/21378532?refer=intelligentunit),[DQN5](https://zhuanlan.zhihu.com/p/21421729). A more pragmatical tutorial with pseudo-codes for Q-learning, [this](https://www.analyticsvidhya.com/blog/2019/04/introduction-deep-q-learning-python/) article would be helpful. The main focus for this blog is placed on the review of trading-bot, which is a python lib for applying DQN in trading, source is [here](https://github.com/pskrunner14/trading-bot).

## Learning Environment for RL in trading
- Action at each timestep t:
  - buy, sell or hold
- State at each timestep t:
  - Ideally: current market/stock status at timestep t (OHLCV Data)
  - Trading-bot: sigmoid wrapped intraday Stock Close Price Difference 
    - which is Close(t)-Close(t-1)
    - Size: 10 which is the oberserve window size
    - I dont understand reasons, will keep digging
- Reward:
  - If Action is Sell: 
    - Reward = Sell Price-Buy Price 
  - If Action is Buy or Hold
    - Reward = 0
- Policy:
  - The Decision to take action based on current state 
  - approximate Q-function policy with DNN:
    - Keras MLP with Huber Loss
    - all Dense Layers: 10-128-128-256-256-3
    - Input: Current State
      - Size: 10
    - Output: Action Probabilities
      - Size: 3

## Hyperparameter
- Batch Size: 
  - the size of how much we interact with environment to collect enough data(state,action,reward)
  - default:32
  - fit the policy model DNN after each data collection
- Window Size:
  - determines the state size
  - look-back period from current timestep t
  - default: 32
  - In this case is the stock price from t-10 to t
- Episode:
  - equivalent to Epoch in normal Deep Learning
  - Purpose is to refine model during the retraining
  - default: 50

## Main Loop pseudocode
```
Foreach Episode
  Foreach timestep t in range(data)
    // training
    state = get_current_state(t) //look back 10 days, collect data, pass through sigmoid
    next_state = get_current_state(t+1) //from tomorrow look back 10 days, collect data, pass through sigmoid

    action = get_current_action() //random sample from action size(3) for buy(1),sell(2),hold(3), return buy(1) for the initial
    reward = get_reward() //based on action, calculate reward:0 or sell_price-buy_price
    memory = save_status_in_memory() //save: state,action,reward,next_state,done

    if len(memory)>batch_size:
      train_policy_dnn(batch_size) //sample batch_size dataset from memory, prepare the dataset X,Y, then fit dnn with X,Y

```

## Pseudocode for DNN Training
```
def train_policy_dnn():
  """
  target:
    "assumed" ground truth of Q-value, 
  """

  model = keras_4_layers_model()
  mini_batch = sample_memory(batch_size) //sample raw dataset

  //prepare data
  for state,action,reward,next_state,done in mini_batch:
    if done:
      target = reward //last instance
    else:
      target = reward + gamma*np.amax(model.predict(next_state)) //Q-value, the maximal value of choosing proper action

    q_values = model.predict(state) //get current prediction
    q_values[action] = target //correct the output of selected action with ground-truth(future info,next state)

    X_train.append(state)
    Y_train.append(q_values)

  loss = model.fit(X_train,Y_train,epoch=1).history["loss"]

  return loss


```

## Notes
Codes above deal only with one single DQN scenario. After the initial paper for introducing DQN, ppl have developed T-DQN and Double-DQN which are also implemented in trading-bot, but not discussed here for now. I will discuss them in the future.

The fundamental notion of these 2 new DQN is called "model copy" or "model transfer". That means we only update (weights copy) the **target_model** for a given period, before that we used the previous model(which we believe is still valid for current context) to calculate the ground-truth.

```
def train_policy_dnn_tdqn():
  """
  target:
    "assumed" ground truth of Q-value, 
  """

  model = keras_4_layers_model()
  tárget_mdoel = model.copy()

  mini_batch = sample_memory(batch_size) //sample raw dataset

  if n_iter % reset_count == 0:
    tárget_mdoel.weights = model.weights

  //prepare data
  for state,action,reward,next_state,done in mini_batch:
    if done:
      target = reward //last instance
    else:
      target = reward + gamma*np.amax(target_model.predict(next_state)) //we use here target_model to update the ground-truth

    q_values = model.predict(state) //get current prediction
    q_values[action] = target //correct the output of selected action with ground-truth(future info,next state)

    X_train.append(state)
    Y_train.append(q_values)

  loss = model.fit(X_train,Y_train,epoch=1).history["loss"]

  return loss


```

<!-- ## Value Function

- Target: 
  - a decision problem: how to use X Dollars to maximize its **reward**
- Policy(Decision Function):
  - simple policy: based only on capital X
```
  if X > 50k:
    invest in stock market
  else if X > 100k
    invest in immobilien market
  else:
    invest in books to learn something
``` 
  - better policy: based on capital X and intrinsic **value**

```
  invest in policy x, which x has estimated maximal reward.

  Expected x return:
  if x = invest in stock market:
    estimated reward = -100 (because of bearish market)
  else if x = invest in immobilien market & Capital X > 100k:
    estimated reward = +500 (because the immo bubble still exists)
  else if x = invest in books & Capital X < 100k:
    estimated reward = +1000 (because u learn somthing!)

```

## Bellman Equation
- Target:
  - How to quantitatively estimate the **value function**?

- Return is cumulative reward 
![Cumulative Reward](https://www.zhihu.com/equation?tex=G_t+%3D+R_%7Bt%2B1%7D+%2B+%5Clambda+R_%7Bt%2B2%7D+%2B+...+%3D+%5Csum_%7Bk%3D0%7D%5E%5Cinfty%5Clambda%5EkR_%7Bt%2Bk%2B1%7D)

We have discounted factor lambda before each reward.

- Definition:
  - **Value Function** v(s) is the Estimation of **Return** at a given state s
  ![iterative form of v(s)](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D%0A+v%28s%29+%26+%3D+%5Cmathbb+E%5BG_t%7CS_t+%3D+s%5D+%5C%5C%5C%5C%0A++++++%26+%3D+%5Cmathbb+E%5BR_%7Bt%2B1%7D%2B%5Clambda+R_%7Bt%2B2%7D+%2B+%5Clambda+%5E2R_%7Bt%2B3%7D+%2B+...%7CS_t+%3D+s%5D+%5C%5C%5C%5C+%0A++++++%26+%3D+%5Cmathbb+E%5BR_%7Bt%2B1%7D%2B%5Clambda+%28R_%7Bt%2B2%7D+%2B+%5Clambda+R_%7Bt%2B3%7D+%2B+...%29%7CS_t+%3D+s%5D+%5C%5C%5C%5C%0A++++++%26+%3D+%5Cmathbb+E%5BR_%7Bt%2B1%7D+%2B+%5Clambda+G_%7Bt%2B1%7D%7CS_t+%3D+s%5D+%5C%5C%5C%5C+%0A++++++%26+%3D+%5Cmathbb+E%5BR_%7Bt%2B1%7D+%2B+%5Clambda+v%28S_%7Bt%2B1%7D%29%7CS_t+%3D+s%5D%0A%5Cend%7Balign%7D)

  ![]()


## Action-Value Function - Extension from Bellman Equation
- Definition:
  - The **Action-Value Function** Q(s,a) is the Estimation of **Return**/Cumulative **Reward** at a given state and an action
  ![Action-Value Function](https://www.zhihu.com/equation?tex=%5Cbegin%7Balign%7D%0AQ%5E%5Cpi%28s%2Ca%29+%26+%3D++%5Cmathbb+E%5Br_%7Bt%2B1%7D+%2B+%5Clambda+r_%7Bt%2B2%7D+%2B+%5Clambda%5E2r_%7Bt%2B3%7D+%2B+...+%7Cs%2Ca%5D+%5C%5C%5C%5C%0A%26+%3D+%5Cmathbb+E_%7Bs%5E%5Cprime%7D%5Br%2B%5Clambda+Q%5E%5Cpi%28s%5E%5Cprime%2Ca%5E%5Cprime%29%7Cs%2Ca%5D%0A%5Cend%7Balign%7D)
- Difference
  -   -->