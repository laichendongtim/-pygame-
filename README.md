# 利用pygame开发飞机大战游戏

## 一、前言

 作为python学习的练手项目之一，飞机大战可谓是经典案例了。内容丰富有趣，难度适中，适合初学者作为学习一定阶段的检测工具。网上的资料有很多，但是我还是想把自己做这个项目时候的经验分享给大家。也算是一个总结，毕竟稳固而知新。

## 二、分析模型

  我们如果没有框架来构造一个项目的时候，需要从头开始考虑这个项目结构。将整个难以实现的目标拆分成许多可以实现的小目标。

### 1、游戏结构

拿飞机大战举例，作为一个游戏，它分为两个部分，一个是游戏的初始化，另一个是游戏的循环。

|    游戏初始化    |     游戏循环     |
| :--------------: | :--------------: |
|   设置游戏窗口   |   设置刷新帧率   |
| 绘制图像初始位置 |   检测用户交互   |
|   设置游戏时钟   | 更新所有图像位置 |
|                  |   更新屏幕显示   |

游戏初始化不难理解，一个游戏有基本的界面，必定涉及到初始化的显示问题，而飞机大战游戏的初始化，就包含了其中三个内容：设置游戏窗口、绘制图像初始位置以及设置游戏时钟

游戏循环呢，相当于一个程序正在运行，我们想要他一直正常运行下去，就必须要设定循环，使之不断运行，当外界条件触发，结束运行。说白了，玩游戏也是运行程序的一个过程。其中包含设置刷新帧率、检测用户交互、更新所有图像位置以及更新屏幕显示。要说明的是，刷新帧率这个词。游戏中的动画效果，本质上是快速的在屏幕上绘制图像，产生连贯的视觉效果，每次绘制的结果被称为`帧`

### 2、游戏内容

飞机大战主要目标（对象）有哪些？背景、我方战机、敌方战机、子弹。

看下来其实并没有多复杂，创建上述对象的过程无非是利用 pygame中自带的类来创建相应的对象。本次用到的是Sprite类。我们看一下他的定义

```python
class Sprite(object):
    """simple base class for visible game objects

    pygame.sprite.Sprite(*groups): return Sprite

    The base class for visible game objects. Derived classes will want to
    override the Sprite.update() method and assign Sprite.image and Sprite.rect
    attributes.  The initializer can accept any number of Group instances that
    the Sprite will become a member of.

    When subclassing the Sprite class, be sure to call the base initializer
    before adding the Sprite to Groups.

    """

    def __init__(self, *groups):
        self.__g = {} # The groups the sprite is in
        if groups:
            self.add(*groups)

    def add(self, *groups):
        """add the sprite to groups

        Sprite.add(*groups): return None

        Any number of Group instances can be passed as arguments. The
        Sprite will be added to the Groups it is not already a member of.

        """
        has = self.__g.__contains__
        for group in groups:
            if hasattr(group, '_spritegroup'):
                if not has(group):
                    group.add_internal(self)
                    self.add_internal(group)
            else:
                self.add(*group)

    def remove(self, *groups):
        """remove the sprite from groups

        Sprite.remove(*groups): return None

        Any number of Group instances can be passed as arguments. The Sprite
        will be removed from the Groups it is currently a member of.

        """
        has = self.__g.__contains__
        for group in groups:
            if hasattr(group, '_spritegroup'):
                if has(group):
                    group.remove_internal(self)
                    self.remove_internal(group)
            else:
                self.remove(*group)

    def add_internal(self, group):
        self.__g[group] = 0

    def remove_internal(self, group):
        del self.__g[group]

    def update(self, *args):
        """method to control sprite behavior

        Sprite.update(*args):

        The default implementation of this method does nothing; it's just a
        convenient "hook" that you can override. This method is called by
        Group.update() with whatever arguments you give it.

        There is no need to use this method if not using the convenience
        method by the same name in the Group class.

        """
        pass

    def kill(self):
        """remove the Sprite from all Groups

        Sprite.kill(): return None

        The Sprite is removed from all the Groups that contain it. This won't
        change anything about the state of the Sprite. It is possible to
        continue to use the Sprite after this method has been called, including
        adding it to Groups.

        """
        for c in self.__g:
            c.remove_internal(self)
        self.__g.clear()

    def groups(self):
        """list of Groups that contain this Sprite

        Sprite.groups(): return group_list

        Returns a list of all the Groups that contain this Sprite.

        """
        return list(self.__g)

    def alive(self):
        """does the sprite belong to any groups

        Sprite.alive(): return bool

        Returns True when the Sprite belongs to one or more Groups.
        """
        return truth(self.__g)

    def __repr__(self):
        return "<%s sprite(in %d groups)>" % (self.__class__.__name__, len(self.__g))
```

可谓是五脏俱全了，该有的函数都有了。我们只需要以此作为父类，派生出相应的子类即可。

## 三、构造函数

### 1.游戏函数框架构造

```python
class PlaneGame(object):
    """main"""
    def __init__(self):
        print("游戏初始化")


        self.screen = pygame.display.set_mode(SCREEN_RECT.size)   # create game win 480*700
        self.clock = pygame.time.Clock()                          # creat game clock
        self._creat_sprites()

        pygame.time.set_timer(CREATR_ENEMY_EVENT,1000)
```

首先创建一个 名字为`fly_main.py`的python文件，在其中创建 `PlaneGame`的一个主函数类，我们所有的主要功能都在里面实现。

初始化过程中，在创建这个类的时候 即 `def __init__(self):`中就包含游戏的窗口创建、时钟创建、精灵（sprites）创建和定时器。

然后在代码的下方分别加入以下函数

```python
def _creat_sprites(self):  #创建精灵 ：创建背景、我方战机、敌方战机
def start_game(self):      #开始游戏 ：建立循环 设置刷新帧率刷新帧率、检测用户交互、更新所有图像位置                                      以及更新屏幕显示
def __event_handler(self): #事件监听 ：检测键盘操作
def __check_collide(self): #碰撞检测 ：检测敌机与我方是否碰撞
def __update_sprite(self): #更新绘制精灵组 
def _game_over():          #结束游戏 ：退出循环
```

### 2.游戏对象创建

创建`fly_sprite.py`的文件，在其中创建基于sprite的类的game类

```python
class GameSprite(pygame.sprite.Sprite):
    """飞机大战游戏精灵"""

    def __init__(self, image_name, speed=1):
        # 调用父类初始化方法
        super().__init__()
        # 定义对象属性
        self.image = pygame.image.load(image_name)
        self.rect = self.image.get_rect()
        self.speed = speed

    def update(self):
        self.rect.y += self.speed
```

可以看到初始化过程中，包含载入图像、获取图像位置以及设置图像的运动速度。

图像的压缩包会在文章结尾给出，默认图像的运动速度为1

在此基础上，调用父类，创建以下子类

```python
class BackGround(GameSprite):
    """游戏背景精灵"""
class Enemy(GameSprite):
    """敌机精灵"""
class Hero(GameSprite):
    """"英雄精灵"""
class Bullets(GameSprite):
    """"子弹精灵"""
```

注意一点，`fly_main.py`中使用`fly_sprite.py`的类和对象的时候记得引用 import 导入文件



## 四、代码编写

到目前为止，这个游戏的基本框架已经搭好，剩下的就是编写相关的代码，不断调试了。

注意一点：坐标轴原点是左上角，向下y轴数值增加，x轴向右数值增加。游戏中所有可见的元素都是用矩形区域描述的。（x,y）（宽，高）。

### 1、游戏对象

`fly_sprite.py`文件中，确定游戏区域

```python
SCREEN_RECT = pygame.Rect(0, 0, 480, 700)  # 屏幕大小常量
```

游戏背景精灵：由于所给的图像和上述屏幕大小相比略小，我们需要用两张同样的图片拼接在一起，连续的移动就能使之成为动态的背景

```python
class BackGround(GameSprite):
    """游戏背景精灵"""

    def __init__(self, is_alt=False):
        super().__init__("./images/background.png")
        if is_alt:  # 判断是否是交替图像，如果是就交换位置
            self.rect.y = -self.rect.height

    def update(self):
        super().update()

        if self.rect.y >= SCREEN_RECT.height:  # 判断是否移出屏幕，将图像设置到屏幕的上方
            self.rect.y = -self.rect.height
```

敌机精灵：由于敌机需要从屏幕上方出现，飞到下方消失，所以需要判断是否飞出屏幕，释放内存，不然产生的对象过多，会使得游戏“臃肿”。其次，想要敌机出现的位置不同，调用`random`函数，随机指定在屏幕内的位置。       

```python
class Enemy(GameSprite):
    """敌机精灵"""

    def __init__(self):
        super().__init__("./images/enemy1.png")  # 调用父类方法，指定敌机图片
        self.speed = random.randint(1, 3)
        self.rect.bottom = 0  # 指定敌机初始位置
        max_x = SCREEN_RECT.width - self.rect.width
        self.rect.x = random.randint(0, max_x)

    def update(self):
        super().update()  # 调用父类方法，保持垂直方向飞行
        if self.rect.y >= SCREEN_RECT.height:  # 判断是否飞出，释放内存
            self.kill()
```

英雄精灵：设置fire函数，调用子弹类。由于背景图像向下滚动设置，所有我方战机不用上下移动，只需要左右运动即可。

```python
class Hero(GameSprite):
    """"英雄精灵"""

    def __init__(self):
        super().__init__("./images/me1.png", speed=0)  # 初始速度为禁止
        self.rect.centerx = SCREEN_RECT.centerx
        self.rect.bottom = SCREEN_RECT.bottom - 120

        self.bullets = pygame.sprite.Group()  # 创建子弹精灵组

    def update(self):
        self.rect.x += self.speed  # 水平方向移动
        if self.rect.x < 0:  # 英雄不能离开屏幕
            self.rect.x = 0
        elif self.rect.right > SCREEN_RECT.right:
            self.rect.right = SCREEN_RECT.right

    def fire(self):
        for i in (0, 1, 2):
            bullet = Bullets()

            bullet.rect.bottom = self.rect.y - i * 20  # 设置子弹位置
            bullet.rect.centerx = self.rect.centerx
            self.bullets.add(bullet)
```

子弹精灵：子弹出现的位置，必须是在hero图像的正上方，且运动方式是从下至上。

```python
class Bullets(GameSprite):
    """"子弹精灵"""

    def __init__(self):
        super().__init__("./images/bullet2.png", speed=-2)

    def update(self):
        super().update()
        if self.rect.bottom < 0:  # 判断子弹是否飞出屏幕
            self.kill()
```

### 2、主函数

创建精灵组和更新绘制精灵不再叙述，本质就是调用 创建的精灵类的更新和绘制函数。

这里着重说一下事件监听以及碰撞检测函数

```python
def __event_handler(self):           #事件监听

    for event in pygame.event.get(): #判断是否退出游戏
        if event.type == pygame.QUIT:
            print("退出游戏")
            self._game_over()
        elif event.type==CREATR_ENEMY_EVENT:
            enemy=Enemy()    #创建敌机
            self.enemy_group.add(enemy)

    #使用键盘提供的方法获取键盘按键
    key_pressed=pygame.key.get_pressed()
    if key_pressed[pygame.K_RIGHT]:
        self.hero.speed=2
    elif key_pressed[pygame.K_LEFT]:
        self.hero.speed=-2

    elif key_pressed[pygame.K_SPACE]:#按下空格开火
        self.hero.fire()
    else:
        self.hero.speed=0

def __check_collide(self):    #碰撞检测

    pygame.sprite.groupcollide(self.hero.bullets,self.enemy_group,True,True)  #子弹摧毁敌机
    enemises=pygame.sprite.spritecollide(self.hero,self.enemy_group,True)     #英雄碰撞敌机
    if len(enemises)>0:
        self.hero.kill()
        PlaneGame._game_over()
```

事件检测就是判断是否退出游戏，以及我们当前在键盘上的操作：左右移动以及发射子弹。

碰撞检测就是判断我方英雄是否与敌机相撞，二者之间的坐标重合即为相撞，随即调用game_over函数结束游戏。


整个游戏较为简单，想要尝试添加新功能的小伙伴，可以继续在此基础上增加趣味性。
