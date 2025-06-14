# 基于 Cocos2d-x 的2048小游戏

 - **[可执行文件下载](https://pan.baidu.com/s/1OjnK47FDsOW-wd1cwBxcLQ?pwd=1234)**
 - **[测试视频](https://github.com/Area-Ivy/2048/blob/main/%E6%BC%94%E7%A4%BA%E6%B5%8B%E8%AF%95%E8%A7%86%E9%A2%91.mp4)**

## 游戏功能

### 基础功能
- **基础方块**：方块以不同颜色代替传统的数字显示，生成的方块对应的数字为 2 和 4，随机出现在棋盘上的空位。
- **合成规则**：相同数字的方块在滑动时若相撞，则合成一个新的方块，数值为原方块之和。
- **权值计算**：每合成一个新数字，玩家得分为新方块的数字值。

### 音效与音乐
- **背景音乐**：支持背景音乐，以营造舒适的游戏氛围。
- **音效**：方块合成时提供音效。

### 基本界面
- **开始界面**：包含游戏的启动界面，可选择继续游戏或新游戏。
- **游戏界面**：包含游戏界面，显示当前分数和最高分。
- **撤回功能**：支持撤回上一步操作。
- **退出功能**：支持退出游戏。

### 状态恢复
- **恢复功能**：支持中途退出时记录当前状态，下次进入时可以选择继续上一次游戏或重新开始。

### 排行榜
- **排行榜**：支持本地排行榜。

## 方块颜色对应关系

| 数字  | 颜色名称            | RGB 颜色值    | HEX 颜色值 |
|-------|---------------------|---------------|------------|
| 2     | 淡黄色             | (247, 213, 97) | #F7D561    |
| 4     | 浅绿色             | (166, 232, 103)| #A6E867    |
| 8     | 绿色               | (87, 212, 154) | #57D49A    |
| 16    | 青绿色             | (19, 181, 177) | #13B5B1    |
| 32    | 蓝色               | (68, 138, 202) | #448ACA    |
| 64    | 紫色               | (200, 97, 234) | #C863EA    |
| 128   | 粉红色             | (225, 115, 181)| #E173B5    |
| 256   | 鲜艳的粉红色       | (238, 100, 141)| #EE648D    |
| 512   | 橙色               | (243, 157, 79) | #F39D4F    |
| 1024  | 橙色               | (245, 124, 78) | #F57C4E    |
| 2048  | 鲜红色             | (246, 76, 20)  | #F64C14    |


# 核心实现 GameScene 

1. 场景创建 (`createScene` 方法)

```cpp
Scene* GameScene::createScene()
{
    auto scene = Scene::create(); // 创建一个新的场景对象
    auto layer = GameScene::getInstance(); // 获取 GameScene 单例
    scene->addChild(layer); // 将 GameScene 单例作为子节点添加到场景中
    return scene; // 返回场景对象
}
```
目的：该方法用于创建一个新的 Scene 对象，并将 GameScene 层作为子节点添加到该场景中，最终返回场景对象。
核心实现：
Scene::create()：创建一个新的场景对象。
GameScene::getInstance()：获取 GameScene 单例对象，确保只存在一个 GameScene 实例。
scene->addChild(layer)：将层 GameScene 添加到场景中。

2. 获取 GameScene 单例 (getInstance 方法)

```cpp

static GameScene* _gameScene;

GameScene* GameScene::getInstance()
{
    if (!_gameScene) // 如果单例对象为空
        _gameScene = GameScene::create(); // 创建新的 GameScene 对象
    return _gameScene; // 返回单例对象
}
```
目的：实现 GameScene 的单例模式，确保全局只有一个 GameScene 实例。
核心实现：
使用静态变量 _gameScene 来存储 GameScene 单例。
如果单例为空（即首次调用），通过 GameScene::create() 创建一个新的实例。
返回单例对象 _gameScene。

3. 初始化方法 (init 方法)

```cpp

bool GameScene::init()
{
    if (!Layer::init()) // 检查父类 Layer 初始化是否成功
        return false; // 如果失败，返回 false
    
    // 播放背景音乐，参数 true 表示循环播放
    AudioEngine::play2d("game_bg.mp3", true);

    // 添加背景颜色层，设置为浅灰色 (242, 242, 242) 不透明
    this->addChild(LayerColor::create(Color4B(242, 242, 242, 255))); 

    // 获取并添加工具层(GameTool)
    auto tool = GameTool::getInstance();
    this->addChild(tool);

    // 获取并添加菜单按钮层(GameMenuLayer)
    auto menuBtnLayer = GameMenuLayer::getInstance();
    this->addChild(menuBtnLayer);

    // 获取并添加游戏层(GameLayer)
    auto gameLayer = GameLayer::getInstance();
    this->addChild(gameLayer);

    // 创建排行榜层(SetMenu)，默认不可见
    auto setLayer = SetMenu::create();
    setLayer->setName("setlayer"); // 设置层的名称为 "setlayer"
    setLayer->setVisible(false); // 设置为不可见
    this->addChild(setLayer);
    
    // 创建开始菜单层(StartLaye)
    auto startLayer = StartLayer::create();
    this->addChild(startLayer);
    
    return true; // 初始化成功，返回 true
}
```
目的：实现 GameScene 的初始化工作，包括设置背景颜色、播放背景音乐、添加游戏层、工具层和菜单层等。

核心实现：

父类初始化检查：

if (!Layer::init())：调用父类的 init 方法并检查返回值。如果失败，初始化 GameScene 失败，返回 false。
背景音乐播放：

AudioEngine::play2d("game_bg.mp3", true)：播放游戏的背景音乐，第二个参数 true 表示循环播放。
添加背景层：

LayerColor::create(Color4B(242, 242, 242, 255))：创建一个浅灰色的背景层，并将其添加到 GameScene 中。
添加游戏组件层：

GameTool::getInstance()：获取游戏工具层（如分数、道具等），并将其添加到场景中。
GameMenuLayer::getInstance()：获取游戏菜单层（包含暂停、设置按钮等），并将其添加到场景中。
GameLayer::getInstance()：获取实际的游戏层，包含游戏逻辑和画面元素，添加到场景中。
添加设置菜单层：

SetMenu::create()：创建排行榜层，该层默认不可见。
setLayer->setName("setlayer")：给设置层命名为 "setlayer"。
setLayer->setVisible(false)：设置该层不可见，直到需要时才显示。

StartLayer::create()：创建开始菜单层，包含新游戏和继续游戏两种选择按钮。
   

## 核心玩法 GameLayer

1. 类结构与初始化
GameLayer 是游戏的主层，负责游戏的核心逻辑和 UI 渲染。每个游戏格子 (Grid) 存储了它的分数，并能响应用户的操作。
游戏格子的状态通过 _grids（一个 4x4 的二维数组）来维护，每个位置可以是一个 Grid 对象，也可能为空（表示没有数字）。
init() 方法负责初始化游戏界面，包括背景和 4x4 的格子。

2. 生成随机格子
游戏开始时，通过 randGenGrid() 随机生成 2 个格子，并随机选择它们的位置和初始值。
randGenGrid() 方法负责生成一个新的格子，并将它放到空白的位置。

3. 保存与恢复游戏状态
游戏支持撤销操作（undoGame()），通过 saveLastGrids() 和 recoverLastGrids() 方法来保存和恢复上一次的游戏状态。
游戏状态被保存在 _lastGrids 中，每当用户移动格子或合并格子时，都会更新这个数组以记录当前的状态。

4. 用户操作与移动
用户通过触摸屏幕进行操作，onTouchBegan、onTouchMoved、onTouchEnded 这些回调函数处理触摸事件。
根据用户的滑动方向，游戏会判断是向左、右、上或下移动，并执行相应的动作（moveToLeft()、moveToRight()、moveToTop()、moveToBottom()）。
每次移动时，游戏格子会被移到目标位置。如果相邻的格子值相同，则会合并（即数值翻倍），并将合并的格子移除。

5. 判断游戏结束
游戏结束的判断在 ifOver() 方法中进行，检查是否有空格子或者相邻的格子能合并。如果所有格子都被填满，且没有可以合并的格子，游戏结束。
如果游戏结束，会显示一个提示框，显示 "Game Over!"。

6. 移动与合并逻辑
游戏的核心在于格子的移动与合并，每次用户操作都会触发一次移动。格子的移动通过 moveGrid()、moveOnly()、moveAndClear() 和 moveAndUpdate() 方法来实现。
moveGrid() 方法会将一个格子移动到目标位置。如果可以合并，则将两个相同的格子合并并更新分数。
moveOnly() 只负责移动格子而不合并。
moveAndClear() 会清除移动前的格子位置，完成合并后更新新格子的值。
moveAndUpdate() 用于在合并后生成一个新的格子，并更新显示。

7. 分数与音效
每次合并格子时，分数会按照合并的数字进行累加。分数由 GameTool 负责管理。
每次合并格子时会播放一个音效

8. 其他辅助功能
游戏支持重新开始（restartGame()）和撤销操作（undoGame()），使玩家可以重新开始游戏或撤销上一次的操作。
clearGrids() 和 clearLastGrids() 用于清理格子数据，恢复到初始状态。


## 游戏菜单 GameMenuLayer

1. 类结构与初始化
GameMenuLayer 是游戏菜单的主界面，它继承自 Layer，并通过 create() 方法返回一个单例实例（getInstance()）。这个菜单层负责创建并显示几个按钮，用于操作游戏的不同功能。

	在 init() 方法中，初始化了菜单的各个按钮，并将它们布局在屏幕上。

2. 按钮布局与创建
在 init() 中，三个主要按钮被创建并添加到菜单中，每个按钮对应一个功能：

	重置游戏按钮：用于重新开始游戏。
	resetGameFun 方法负责调用 GameLayer 中的 restartGame() 方法，重新开始游戏。
	按钮位置和大小是由 resetBg 和 resetmenu 定义的，按钮的文本为 "重新开始"。
	
	排行榜按钮：用于显示或隐藏设置排行榜。
	setGameFun 方法会切换设置排行榜的可见性。
	当按钮被点击时，setmenu 会根据当前状态显示或隐藏排行榜菜单（SetMenu）。
	按钮的位置和大小由 setBg 和 setmenu 定义，按钮文本为 "排行榜"。
	
	撤销按钮：用于撤销到上一个游戏状态。
	undoGameFun 方法会调用 GameLayer 中的 undoGame() 方法，恢复到游戏的上一个状态。

3. UI 布局
这些按钮（resetmenu、undomenu 和 setmenu）被放置到不同的位置，分别创建了背景层 (LayerColor) 来放置按钮并设置它们的大小和位置。
所有的按钮被添加到一个 Menu 中，Menu 用来管理多个按钮，并设置它们的点击事件。

4. 按钮回调与功能实现
每个按钮都有一个回调函数，在点击按钮时会调用相应的函数：

	resetGameFun(Ref* ref)：
	当点击“重新开始”按钮时，调用 GameLayer::getInstance()->restartGame() 来重置游戏的状态。
	setGameFun(Ref* ref)：
	当点击“设置”按钮时，检查设置菜单（setmenu）是否可见，如果可见则隐藏它，否则显示它。
	undoGameFun(Ref* ref)：
	当点击“撤销”按钮时，调用 GameLayer::getInstance()->undoGame() 恢复到游戏的上一个状态。

5. UI 的创建与显示
所有的按钮和背景层都添加到 GameMenuLayer 中，并通过 Menu::create() 创建一个菜单容器，将所有按钮放在一起。
menu->setPosition(0, 0) 确保按钮菜单的位置在父容器的 (0, 0) 位置上。

6. 按钮位置与样式
每个按钮的背景层（LayerColor）设置了不同的颜色 (Color4B(143, 122, 101, 255)) 和大小。
按钮的文本是通过 MenuButton::create() 创建的，menuButton->getMenuItem() 方法负责获取一个包含文本的菜单项，并设置它们的回调。

## 分数菜单 GameTool

1. 类结构与初始化
GameTool 是游戏菜单的主界面，它继承自 Layer，并通过 create() 方法返回一个单例实例（getInstance()）。这个菜单层负责创建并显示分数。

	在 init() 方法中，初始化了菜单2048标题标签和分数显示标签，并将它们布局在屏幕上。

2. 分数标签布局与创建
在 initScore() 中，两个分数标签被创建并添加到菜单中，每个标签对应一个显示功能：

	分数标签：用于显示当前分数。
	当前分数通过getScore()获取，并通过updateScore(int addScore)更新分数并检测是否需要更新最佳分数，通过setScore(int score)实时更新显示。
	
	最佳分数标签：用于显示最佳分数。
	通过updateScore(int addScore)更新分数并检测是否需要更新最佳分数，需要更新则通过updateBestScore()更新最佳分数，并通过setBestScore(int bestScore)更新显示。
	
## Grid 格子类实现

Grid 类核心实现文档
Grid 类是用于管理和显示游戏中的网格（格子）元素，主要用于类似 2048 游戏的实现中。它负责管理每个格子的状态、位置、大小、显示内容以及动画效果。下面是该类的核心实现和功能的详细说明。

1. 属性声明
	_value：该格子的值，决定了格子的显示内容。
	_bg：格子的背景，使用 LayerColor 实现，显示不同颜色。

2. 方法概述
	2.1 G2U(const char* gb2312)
	将 gb2312 编码的字符串转换为 UTF-8 编码。用于处理中文字符的显示。

	2.2 init()
	初始化方法，设置格子尺寸、背景等元素，并调用 updateBg() 来更新格子的外观。

	2.3 compareTo(Grid* grid)
	比较当前格子的值与另一个格子的值，返回是否相等。

	2.4 initValue(int value)
	初始化格子的值，并调用 updateBg() 更新格子的背景和显示内容。

	2.5initValue(int value, int row, int column)
	除了初始化值外，还设置格子的位置（根据行列坐标）。

	2.6 setLocalPosition(int row, int column)
	设置格子的屏幕位置，位置计算公式是 (column * 73 + 8, row * 73 + 8)，格子的大小为 65x65，边距为 8。

	2.7 initAction()
	格子初始化动画。执行缩放和淡入效果，使格子从无到有的显示出来。

	2.8 moveOnly(int targetRow, int targetColumn)
	仅移动格子到目标位置，不进行其他动画效果。

	2.9 moveAndClear(int targetRow, int targetColumn)
	格子移动到目标位置并同时淡出，然后从父节点中移除，执行“清除”动作。

	2.10 moveAndUpdate()
	执行移动动画、缩放动画，并且更新格子的值和显示内容。
	在动画结束后通过 CallFunc 让格子显示出来。

	2.11 updateBg()
	根据当前格子的 _value 更新颜色。

3. 动画效果
initAction()：通过 ScaleTo 和 FadeIn 动画让格子从小到大，并渐显出现。

	moveAndClear()：通过 MoveTo 和 FadeOut 动画将格子移动并隐藏，最后从父节点移除。

	moveAndUpdate()：通过缩放和淡入的方式，使格子在移动后更新显示并重新显示出来。

4. 颜色与字体设置
根据 _value 的不同，格子的颜色会发生变化。
比如，_value == 0 时，颜色为 #F7D561；而 _value == 1 时，颜色为 #A6E867。

5. 总结
Grid 类是该游戏实现中的一个基础组件，用于管理每个格子的显示、移动和动画效果。通过灵活的动画和动态更新，它能实时反映游戏中的格子变化。每个格子不仅仅是一个简单的矩形，它还承载着分数、背景颜色、动画效果等多种信息，是游戏界面中不可或缺的元素。


## 排行榜 SetMenu

 通过加载分数vector渲染每个分数的排名情况
 ``` c++
  for (size_t i = 0; i < scores.size(); ++i)
    {
        logStr += Value(scores[i]).asString();
        auto rankLabel = Label::createWithSystemFont(Value(scores[i]).asString(), "Arial", 16);
        rankLabel->setAnchorPoint(Vec2(0, 0.5f));
        rankLabel->setPosition(Vec2(50, yStart - i * offset));
        this->addChild(rankLabel);
        if (i != scores.size() - 1)
            logStr += ", ";
    }
```


## 数据存储 DataConf

DataConf 类核心实现文档
DataConf 类用于管理游戏的存档和配置数据，负责将游戏的状态（如分数、格子数据、游戏类型等）保存到本地，以便玩家下次启动游戏时能够恢复上次的状态。它还提供了单例模式的实现，以确保在整个游戏过程中只有一个 DataConf 实例。

1. 类成员与方法概述
1.1 单例模式
```cpp

static DataConf* instance;
DataConf* DataConf::getInstance()
{
    if (instance == nullptr)
        instance = create();

    return instance;
}
```
instance：静态指针，指向唯一的 DataConf 实例。
getInstance()：用于获取 DataConf 的单例。如果实例尚未创建，则调用 create() 创建新的实例。

1.2 初始化方法
```cpp
bool DataConf::init()
{
    return true;
}
```
init()：初始化方法，当前没有任何操作，直接返回 true。可扩展为初始化配置或加载其他数据。

1.3 数据存储与保存
```cpp
    auto f = UserDefault::getInstance();
    f->setIntegerForKey(Value(type).asString().append("score").c_str(), GameTool::getInstance()->getScore());
    f->setIntegerForKey(Value(type).asString().append("best_score").c_str(), GameTool::getInstance()->getBestScore());
    f->setIntegerForKey("type", type);
    f->setBoolForKey(Value(type).asString().append("exits").c_str(), true);
    GameTool::getInstance()->saveScoresToLocal();
    f->flush();
   ```
将当前游戏状态保存到本地存储。具体内容包括：

保存当前得分和最佳得分：通过 GameTool::getInstance()->getScore() 和 GameTool::getInstance()->getBestScore() 获取当前得分和最佳得分，并使用 UserDefault 存储。

保存游戏类型：通过 f->setIntegerForKey("type", type) 保存当前游戏模式类型（如经典模式、颜色模式等）。

保存当前游戏是否存在：使用 f->setBoolForKey(... "exits") 标记游戏是否存在（即玩家是否进行了游戏）。

将当前得分列表保存到本地：调用 GameTool::getInstance()->saveScoresToLocal()，此方法负责将历史得分保存到本地存储。

f->flush()：确保所有保存的数据立即写入磁盘。


1.4 保存每个格子的状态
```cpp
Grid* temp;
int value;
for (int i = 0; i < 4; i++)
{
    for (int j = 0; j < 4; j++)
    {
        temp = GameLayer::getInstance()->_grids[i][j];
        if (temp == nullptr)
            value = -1;
        else
            value = temp->getScoreValue();
        f->setIntegerForKey(Value(type * 3).asString().append(Value(100 + i * 4 + j).asString()).c_str(), value);
        f->setIntegerForKey(Value(type * 3).asString().append(Value(i * 4 + j).asString()).c_str(), GameLayer::getInstance()->_lastGrids[i][j]);
    }
}
```
格子数据保存：循环遍历 4x4 的游戏格子，获取每个格子的值（value）并保存。
如果当前格子为空（temp == nullptr），则将其值设置为 -1。
否则，通过 temp->getScoreValue() 获取当前格子的得分，并保存到本地。

使用 UserDefault 存储格子的状态：使用一个带有唯一键的存储形式保存每个格子的值和之前的状态。
格子的值通过键名 "type* 3 + 100 + i * 4 + j" 保存。
格子之前的状态（上一步的状态）通过键名 "type* 3 + i * 4 + j" 保存。

flush()：每次更新后，调用 flush() 确保所有修改的数据被写入磁盘。


2. 总结
DataConf 类负责游戏数据的存取，主要包括：
游戏状态保存：保存当前得分、最佳得分、游戏模式和格子状态等信息。

	单例模式：确保只有一个 DataConf 实例，便于管理游戏的全局配置。

	格子状态保存：存储每个格子的值和状态，以便在游戏重启时恢复。

##  开始界面 StartLayer
StartLayer 类核心实现文档
StartLayer 类用于显示开始菜单，为玩家提供“新游戏”和“继续游戏”两种选择。

1.界面创建
```cpp
 // 获取屏幕尺寸
 Size visibleSize = Director::getInstance()->getVisibleSize();
 Vec2 origin = Director::getInstance()->getVisibleOrigin();

 // 创建黑色背景
 auto background = LayerColor::create(Color4B(0, 0, 0, 180));  // 黑色背景，透明度180
 background->setPosition(Vec2::ZERO);  // 将背景层位置设置为原点
 this->addChild(background);

 // 创建“新游戏”按钮
 auto newGameItem = MenuItemLabel::create(Label::createWithTTF("New Game", "fonts/arial.ttf", 24),
     CC_CALLBACK_1(StartLayer::onNewGameClicked, this));
 newGameItem->setPosition(Vec2(visibleSize.width / 2, visibleSize.height / 2 + 50));

 // 创建“继续游戏”按钮
 auto continueGameItem = MenuItemLabel::create(Label::createWithTTF("Continue Game", "fonts/arial.ttf", 24),
     CC_CALLBACK_1(StartLayer::onContinueGameClicked, this));
 continueGameItem->setPosition(Vec2(visibleSize.width / 2, visibleSize.height / 2 - 50));

 // 将按钮放入菜单中
 auto menu = Menu::create(newGameItem, continueGameItem, nullptr);
 menu->setPosition(Vec2::ZERO);
 this->addChild(menu);

 return true;
```
创建开始菜单界面和按钮。具体内容包括：
创建黑色背景：通过 LayerColor::create(Color4B(0, 0, 0, 180))创建半透明黑色背景，并添加到层中。

创建“新游戏”按钮：通过MenuItemLabel::create(Label::createWithTTF("New Game", "fonts/arial.ttf", 24)创建按钮，并通过CC_CALLBACK_1设置回调操作。

创建“继续游戏”按钮：使用 MenuItemLabel::create(Label::createWithTTF("Continue Game", "fonts/arial.ttf", 24)创建按钮，并通过CC_CALLBACK_1设置回调操作。

将按钮放入菜单中：通过Menu::create(newGameItem, continueGameItem, nullptr)实现。

2.按下按钮执行的操作
```cpp
// 点击“新游戏”按钮后的处理函数
void StartLayer::onNewGameClicked(Ref* sender) {
    GameLayer::getInstance()->restartGame();
    // 隐藏当前 UI 层
    this->setVisible(false);
    log("reset game");
}

// 点击“继续游戏”按钮后的处理函数
void StartLayer::onContinueGameClicked(Ref* sender)
{
    // 隐藏当前 UI 层
    this->setVisible(false);
}
```
选择“新游戏”时，执行restartGame()，设置当前UI层(StartLayer)为不可见。

选择“继续游戏”时，设置当前UI层(StartLayer)为不可见，无需执行额外操作。

