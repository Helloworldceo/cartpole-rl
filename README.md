# Cart-Pole RL Lab

The same cart-pole balancing problem as [cartpole-control](../cartpole-control) — but this time nothing is told how the physics works. A tabular **Q-learning** agent starts knowing nothing and has to discover a balancing policy purely from trial and error, guided by nothing more than +1 reward per surviving timestep.

No install, no build step, no dependencies. It's a single HTML file — open it and it runs.

![Initial view](screenshots/1-initial.png)

## Quick start

1. Download or clone this repo.
2. Open `index.html` in any modern browser.
3. Click **Train 1,000** and watch the episode counter climb and the learning curve start to trend upward.
4. When training finishes, click **▶ Watch trained agent** to see the current policy attempt a live balancing run.

## Step by step

### 1. Train the agent

The environment matches the classic **CartPole-v1** benchmark exactly: cart mass 1.0 kg, pole mass 0.1 kg, pole half-length 0.5 m, ±10 N bang-bang force, 0.02 s timestep. An episode ends when the cart leaves ±2.4 m or the pole tips past ±12°, and reward is +1 per surviving step, capped at 500.

Click **Train 100**, **Train 1,000**, **Train 5,000**, or **Train 20,000** to run that many episodes back-to-back. Training is chunked so the UI stays responsive — you'll see the episode counter, current ε (exploration rate), best episode, and rolling average all update live as it goes, along with the learning curve chart.

Because the agent can't perceive the world continuously, its 4-dimensional state (position, velocity, angle, angular velocity) is discretized into bins — adjustable under **State Discretization**. More bins per dimension means finer perception but a larger table to fill in, so it can need *more* training, not less, to reach the same performance.

![A populated learning curve after training](screenshots/2-learning-curve.png)

The raw episode length (thin grey line) is noisy — that's expected, since ε-greedy exploration keeps taking random actions even late in training. The rolling average (purple) is the line to watch; it should trend upward over thousands of episodes.

### 2. Watch it balance

Click **▶ Watch trained agent** to run one real-time episode using the agent's current best-known policy (no exploration — pure greedy action selection). The pole and cart render live, just like the control-systems project, so you can watch it either recover nicely or fall over, depending on how much training it's had.

![Watching a trained agent balance the pole](screenshots/3-watch-agent.png)

### 3. Break it on purpose

Try this to feel the effect of state representation on what's learnable at all:

- Drop the **position x** and **velocity ẋ** bins down to 1 each and retrain from scratch (**Reset Q-table**, then train again). The agent can no longer perceive drift toward the track edge at all — watch the average *get worse*, not better, no matter how long you train. This isn't a slower learner; it's an agent that's blind to information it needs.
- Compare the number of episodes this takes to reach a decent policy against how instantly [cartpole-control](../cartpole-control)'s LQR controller solves the identical physical system with a few matrix operations. That gap — thousands of trial-and-error episodes versus one closed-form linear-algebra solve — is the real, practical cost of not having a model of your environment.

### 4. Tune the learning itself

- **α (learning rate)** — how much each new experience overwrites the old estimate. Too high and learning is noisy; too low and it's painfully slow.
- **γ (discount)** — how much future reward matters relative to immediate reward.
- **ε decay** — how quickly the agent shifts from exploring randomly to exploiting what it's learned.

## How it works

- **Environment & physics**: the exact same verified nonlinear cart-pole RK4 simulation as the control-systems project (`derivatives` / `rk4Step`), just driven by two discrete actions (`+10N` / `-10N`) instead of a continuous force.
- **Discretization**: each of the 4 continuous state variables is binned into buckets; the 4 bin indices combine into one integer "box" index — the same idea behind the classic Barto–Sutton–Anderson approach to making cart-pole tractable for a lookup-table method.
- **Update rule**: standard Q-learning, `Q(s,a) += α · (r + γ · max Q(s′,·) − Q(s,a))`, with ε-greedy action selection.

Everything lives in `index.html` with no external libraries — open it in a text editor to see exactly how it works.

## Related projects

- **[cartpole-control](../cartpole-control)** — the same system, solved instantly with PID and LQR instead of learned by trial and error.
- **[game-theory-lab](../game-theory-lab)** — the Iterated Prisoner's Dilemma and evolutionary game theory.
