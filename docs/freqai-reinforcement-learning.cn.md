# 强化学习

!!! Note "安装大小"
    强化学习依赖包含大型软件包（如 `torch`），在 `./setup.sh -i` 期间需要通过回答 "Do you also want dependencies for freqai-rl (~700mb additional space required) [y/N]?" 问题中选择 "y" 来明确请求安装。
    使用 Docker 的用户应确保使用带有 `_freqairl` 后缀的 Docker 镜像。

## 背景和术语

### 什么是强化学习，为什么 FreqAI 需要它？

强化学习涉及两个重要组件：*智能体*和训练*环境*。在智能体训练期间，智能体逐根 K 线遍历历史数据，始终执行一组动作中的一个：做多入场、做多出场、做空入场、做空出场、中性）。在此训练过程中，环境跟踪这些动作的表现，并根据用户自定义的 `calculate_reward()` 奖励智能体（这里我们提供一个默认奖励供用户在此基础上构建 [详情见此](#creating-a-custom-reward-function)）。奖励用于训练神经网络中的权重。

FreqAI 强化学习实现的第二个重要组件是*状态*信息的使用。状态信息在每一步输入到网络中，包括当前利润、当前仓位和当前交易持续时间。这些用于在训练环境中训练智能体，以及在模拟/实盘中强化智能体（此功能在回测中不可用）。*FreqAI + Freqtrade 非常适合这种强化机制，因为这些信息在实盘部署中可以随时获取。*

强化学习是 FreqAI 的自然发展方向，因为它增加了分类器和回归器无法匹敌的新的适应性和市场反应层。然而，分类器和回归器具有强化学习所没有的优势，例如稳健的预测。训练不当的强化学习智能体可能会找到"作弊"和"技巧"来最大化奖励，但实际上不会赢得任何交易。因此，强化学习比典型的分类器和回归器更复杂，需要更高水平的理解。

### 强化学习接口

在当前框架中，我们旨在通过通用的"预测模型"文件暴露训练环境，该文件是用户继承的 `BaseReinforcementLearner` 对象（例如 `freqai/prediction_models/ReinforcementLearner`）。在此用户类中，强化学习环境可通过 `MyRLEnv` 进行自定义，如[下方所示](#creating-a-custom-reward-function)。

我们设想大多数用户将精力集中在 `calculate_reward()` 函数的创意设计上 [详情见此](#creating-a-custom-reward-function)，而保持环境的其余部分不变。其他用户可能根本不修改环境，他们只会调整配置设置和 FreqAI 中已有的强大特征工程。同时，我们允许高级用户完全创建自己的模型类。

该框架基于 stable_baselines3（torch）和 OpenAI gym 作为基础环境类构建。但总体而言，模型类是很好地隔离的。因此，可以轻松将竞争库集成到现有框架中。对于环境，它继承自 `gym.Env`，这意味着如果要切换到不同的库，需要编写一个全新的环境。

### 重要注意事项

如上所述，智能体在一个人工交易"环境"中被"训练"。在我们的场景中，该环境可能看起来与真正的 Freqtrade 回测环境非常相似，但它*不是*。事实上，强化学习训练环境要简化得多。它不包含任何复杂的策略逻辑，如 `custom_exit`、`custom_stoploss`、杠杆控制等回调。相反，强化学习环境是真实市场的一个非常"原始"的表示，智能体可以自由学习策略（即：止损、止盈等），这由 `calculate_reward()` 强制执行。因此，重要的是要考虑到智能体训练环境与真实世界并不相同。

## 运行强化学习

设置和运行强化学习模型与运行回归器或分类器相同。相同的两个标志 `--freqaimodel` 和 `--strategy` 必须在命令行上定义：

```bash
freqtrade trade --freqaimodel ReinforcementLearner --strategy MyRLStrategy --config config.json
```

其中 `ReinforcementLearner` 将使用 `freqai/prediction_models/ReinforcementLearner` 中的模板 `ReinforcementLearner`（或位于 `user_data/freqaimodels` 中的用户自定义模型）。另一方面，策略遵循与典型回归器相同的基础 [特征工程](freqai-feature-engineering.md)，使用 `feature_engineering_*`。区别在于目标的创建，强化学习不需要目标。但是，FreqAI 需要在动作列中设置一个默认（中性）值：

```python
    def set_freqai_targets(self, dataframe, **kwargs) -> DataFrame:
        """
        *Only functional with FreqAI enabled strategies*
        Required function to set the targets for the model.
        All targets must be prepended with `&` to be recognized by the FreqAI internals.

        More details about feature engineering available:

        https://www.freqtrade.io/en/stable/freqai-feature-engineering

        :param df: strategy dataframe which will receive the targets
        usage example: dataframe["&-target"] = dataframe["close"].shift(-1) / dataframe["close"]
        """
        # For RL, there are no direct targets to set. This is filler (neutral)
        # until the agent sends an action.
        dataframe["&-action"] = 0
        return dataframe
```

大部分函数与典型回归器保持相同，但下面的函数展示了策略必须如何将原始价格数据传递给智能体，以便它在训练环境中可以访问原始 OHLCV 数据：

```python
    def feature_engineering_standard(self, dataframe: DataFrame, **kwargs) -> DataFrame:
        # The following features are necessary for RL models
        dataframe[f"%-raw_close"] = dataframe["close"]
        dataframe[f"%-raw_open"] = dataframe["open"]
        dataframe[f"%-raw_high"] = dataframe["high"]
        dataframe[f"%-raw_low"] = dataframe["low"]
    return dataframe
```

最后，没有明确的"标签"需要创建——而是需要分配 `&-action` 列，该列在 `populate_entry/exit_trends()` 中访问时将包含智能体的动作。在当前示例中，中性动作设为 0。此值应与使用的环境一致。FreqAI 提供两种环境，都使用 0 作为中性动作。

当用户意识到没有标签需要设置后，他们很快就会明白智能体正在做出"自己的"入场和出场决策。这使得策略构建相当简单。入场和出场信号来自智能体，以整数形式——直接用于在策略中决定入场和出场：

```python
    def populate_entry_trend(self, df: DataFrame, metadata: dict) -> DataFrame:

        enter_long_conditions = [df["do_predict"] == 1, df["&-action"] == 1]

        if enter_long_conditions:
            df.loc[
                reduce(lambda x, y: x & y, enter_long_conditions), ["enter_long", "enter_tag"]
            ] = (1, "long")

        enter_short_conditions = [df["do_predict"] == 1, df["&-action"] == 3]

        if enter_short_conditions:
            df.loc[
                reduce(lambda x, y: x & y, enter_short_conditions), ["enter_short", "enter_tag"]
            ] = (1, "short")

        return df

    def populate_exit_trend(self, df: DataFrame, metadata: dict) -> DataFrame:
        exit_long_conditions = [df["do_predict"] == 1, df["&-action"] == 2]
        if exit_long_conditions:
            df.loc[reduce(lambda x, y: x & y, exit_long_conditions), "exit_long"] = 1

        exit_short_conditions = [df["do_predict"] == 1, df["&-action"] == 4]
        if exit_short_conditions:
            df.loc[reduce(lambda x, y: x & y, exit_short_conditions), "exit_short"] = 1

        return df
```

需要注意的是，`&-action` 取决于用户选择使用的环境。上面的示例展示了 5 个动作，其中 0 是中性，1 是做多入场，2 是做多出场，3 是做空入场，4 是做空出场。

## 配置强化学习器

为了配置 `Reinforcement Learner`，`freqai` 配置中必须存在以下字典：

```json
        "rl_config": {
            "train_cycles": 25,
            "add_state_info": true,
            "max_trade_duration_candles": 300,
            "max_training_drawdown_pct": 0.02,
            "cpu_count": 8,
            "model_type": "PPO",
            "policy_type": "MlpPolicy",
            "model_reward_parameters": {
                "rr": 1,
                "profit_aim": 0.025
            }
        }
```

参数详情可在 [此处](freqai-parameter-table.md) 找到，但总体而言，`train_cycles` 决定智能体应在其人工环境中循环遍历 K 线数据多少次来训练模型中的权重。`model_type` 是一个字符串，用于选择 [stable_baselines](https://stable-baselines3.readthedocs.io/en/master/)（外部链接）中的可用模型之一。

!!! Note
    如果你想尝试 `continual_learning`，则应在主 `freqai` 配置字典中将该值设置为 `true`。这将告诉强化学习库从先前模型的最终状态继续训练新模型，而不是每次重新训练时从头开始训练新模型。

!!! Note
    请记住，通用的 `model_training_parameters` 字典应包含特定 `model_type` 的所有模型超参数自定义。例如，`PPO` 的参数可在 [此处](https://stable-baselines3.readthedocs.io/en/master/modules/ppo.html) 找到。

## 创建自定义奖励函数

!!! danger "不适用于生产环境"
    警告！
    Freqtrade 源代码中提供的奖励函数是功能展示，旨在展示/测试尽可能多的环境控制功能。它还被设计为在小型计算机上快速运行。这是一个基准测试，*不*适用于实盘生产。请注意，你需要创建自己的自定义 custom_reward() 函数，或使用其他用户在 Freqtrade 源代码之外构建的模板。

当你开始修改策略和预测模型时，你会很快意识到强化学习器与回归器/分类器之间的一些重要区别。首先，策略不设置目标值（没有标签！）。相反，你在 `MyRLEnv` 类中设置 `calculate_reward()` 函数（见下方）。`prediction_models/ReinforcementLearner.py` 中提供了一个默认的 `calculate_reward()` 来演示创建奖励所需的构建模块，但这*不*是为生产设计的。用户*必须*创建自己的自定义强化学习模型类，或使用 Freqtrade 源代码之外的预构建模型，并将其保存到 `user_data/freqaimodels`。正是在 `calculate_reward()` 中可以表达关于市场的创意理论。例如，你可以在智能体赢利交易时奖励它，在亏损交易时惩罚它。或者，你可能希望在智能体入场交易时奖励它，在持仓时间过长时惩罚它。以下我们展示了这些奖励如何计算的示例：

!!! note "提示"
    最好的奖励函数是连续可微且缩放良好的。换句话说，为罕见事件添加单个大的负惩罚不是一个好主意，神经网络将无法学习该函数。相反，最好为常见事件添加小的负惩罚。这将帮助智能体更快地学习。不仅如此，你还可以通过让奖励/惩罚根据某些线性/指数函数按严重程度缩放来帮助改善奖励/惩罚的连续性。换句话说，你应该随着交易持续时间的增加而缓慢增加惩罚。这比在单个时间点发生单个大惩罚更好。

```python
from freqtrade.freqai.prediction_models.ReinforcementLearner import ReinforcementLearner
from freqtrade.freqai.RL.Base5ActionRLEnv import Actions, Base5ActionRLEnv, Positions


class MyCoolRLModel(ReinforcementLearner):
    """
    User created RL prediction model.

    Save this file to `freqtrade/user_data/freqaimodels`

    then use it with:

    freqtrade trade --freqaimodel MyCoolRLModel --config config.json --strategy SomeCoolStrat

    Here the users can override any of the functions
    available in the `IFreqaiModel` inheritance tree. Most importantly for RL, this
    is where the user overrides `MyRLEnv` (see below), to define custom
    `calculate_reward()` function, or to override any other parts of the environment.

    This class also allows users to override any other part of the IFreqaiModel tree.
    For example, the user can override `def fit()` or `def train()` or `def predict()`
    to take fine-tuned control over these processes.

    Another common override may be `def data_cleaning_predict()` where the user can
    take fine-tuned control over the data handling pipeline.
    """
    class MyRLEnv(Base5ActionRLEnv):
        """
        User made custom environment. This class inherits from BaseEnvironment and gym.Env.
        Users can override any functions from those parent classes. Here is an example
        of a user customized `calculate_reward()` function.

        Warning!
        This is function is a showcase of functionality designed to show as many possible
        environment control features as possible. It is also designed to run quickly
        on small computers. This is a benchmark, it is *not* for live production.
        """
        def calculate_reward(self, action: int) -> float:
            # first, penalize if the action is not valid
            if not self._is_valid(action):
                return -2
            pnl = self.get_unrealized_profit()

            factor = 100

            pair = self.pair.replace(':', '')

            # you can use feature values from dataframe
            # Assumes the shifted RSI indicator has been generated in the strategy.
            rsi_now = self.raw_features[f"%-rsi-period_10_shift-1_{pair}_"
                            f"{self.config['timeframe']}"].iloc[self._current_tick]

            # reward agent for entering trades
            if (action in (Actions.Long_enter.value, Actions.Short_enter.value)
                    and self._position == Positions.Neutral):
                if rsi_now < 40:
                    factor = 40 / rsi_now
                else:
                    factor = 1
                return 25 * factor

            # discourage agent from not entering trades
            if action == Actions.Neutral.value and self._position == Positions.Neutral:
                return -1
            max_trade_duration = self.rl_config.get('max_trade_duration_candles', 300)
            trade_duration = self._current_tick - self._last_trade_tick
            if trade_duration <= max_trade_duration:
                factor *= 1.5
            elif trade_duration > max_trade_duration:
                factor *= 0.5
            # discourage sitting in position
            if self._position in (Positions.Short, Positions.Long) and \
            action == Actions.Neutral.value:
                return -1 * trade_duration / max_trade_duration
            # close long
            if action == Actions.Long_exit.value and self._position == Positions.Long:
                if pnl > self.profit_aim * self.rr:
                    factor *= self.rl_config['model_reward_parameters'].get('win_reward_factor', 2)
                return float(pnl * factor)
            # close short
            if action == Actions.Short_exit.value and self._position == Positions.Short:
                if pnl > self.profit_aim * self.rr:
                    factor *= self.rl_config['model_reward_parameters'].get('win_reward_factor', 2)
                return float(pnl * factor)
            return 0.
```

## 使用 Tensorboard

强化学习模型受益于跟踪训练指标。FreqAI 已集成 Tensorboard，允许用户跟踪所有币种和所有重新训练的训练和评估性能。Tensorboard 通过以下命令激活：

```bash
tensorboard --logdir user_data/models/unique-id
```

其中 `unique-id` 是 `freqai` 配置文件中设置的 `identifier`。此命令必须在单独的 shell 中运行，以便在浏览器中通过 127.0.0.1:6006 查看输出（6006 是 Tensorboard 使用的默认端口）。

![tensorboard](assets/tensorboard.jpg)

## 自定义日志记录

FreqAI 还提供了一个内置的分集摘要记录器 `self.tensorboard_log`，用于向 Tensorboard 日志添加自定义信息。默认情况下，此函数已在环境内的每一步调用一次以记录智能体的动作。在单个分集中为所有步骤累积的所有值都会在每个分集结束时报告，随后完全重置所有指标为 0，为后续分集做准备。

`self.tensorboard_log` 也可以在环境中的任何地方使用，例如，可以将其添加到 `calculate_reward` 函数中以收集有关奖励各部分被调用频率的更详细信息：

```python
    class MyRLEnv(Base5ActionRLEnv):
        """
        User made custom environment. This class inherits from BaseEnvironment and gym.Env.
        Users can override any functions from those parent classes. Here is an example
        of a user customized `calculate_reward()` function.
        """
        def calculate_reward(self, action: int) -> float:
            if not self._is_valid(action):
                self.tensorboard_log("invalid")
                return -2

```

!!! Note
    `self.tensorboard_log()` 函数设计用于仅跟踪递增对象，即训练环境中的事件、动作。如果感兴趣的事件是浮点数，可以将浮点数作为第二个参数传递，例如 `self.tensorboard_log("float_metric1", 0.23)`。在这种情况下，指标值不会递增。

## 选择基础环境

FreqAI 提供三种基础环境：`Base3ActionRLEnvironment`、`Base4ActionEnvironment` 和 `Base5ActionEnvironment`。顾名思义，这些环境是为可以从 3、4 或 5 个动作中选择的智能体定制的。`Base3ActionEnvironment` 是最简单的，智能体可以从持有、做多或做空中选择。此环境也可用于仅做多的机器人（它自动遵循策略中的 `can_short` 标志），其中做多是入场条件，做空是出场条件。而在 `Base4ActionEnvironment` 中，智能体可以做多入场、做空入场、保持中性或退出仓位。最后，在 `Base5ActionEnvironment` 中，智能体具有与 Base4 相同的动作，但不是单个退出动作，而是分离了做多出场和做空出场。环境选择带来的主要变化包括：

* `calculate_reward` 中可用的动作
* 用户策略使用的动作

所有 FreqAI 提供的环境都继承自一个与动作/仓位无关的环境对象 `BaseEnvironment`，该对象包含所有共享逻辑。该架构设计为易于自定义。最简单的自定义是 `calculate_reward()`（详情见[此处](#creating-a-custom-reward-function)）。然而，自定义可以进一步扩展到环境中的任何函数。你可以通过在预测模型文件中的 `MyRLEnv` 中简单地覆盖这些函数来实现。或者对于更高级的自定义，建议创建一个完全新的继承自 `BaseEnvironment` 的环境。

!!! Note
    只有 `Base3ActionRLEnv` 可以进行仅做多的训练/交易（设置用户策略属性 `can_short = False`）。
