# pywin32系列：鼠标、键盘前后台操作、获取文本、指定范围内的颜色值（rgb或16进制）

前后台操作 

获取文本 （可聚焦窗口（可编辑文字）、非可聚焦窗口只能获取到标题（比如歉意生活 - 新订单））

python绑定非可聚焦窗口的句柄后获取该窗口内指定范围的指定颜色值的数量或所有颜色值返回一个列表（判断是否有变化），另外窗口范围是否会受 缩放和分辨率的变化发生变动。

win10系统上用python3.8 通过pyinstaller编译exe缺少api-ms-win-core-path-l1-1-0.dll 导致win7电脑不可用，但是为什么同样的代码win7系统通过pyinstaller编译exe却 win7 win10都可以使用？



## 扩展：

键盘各键对应的ASCII码值

```
ESC键 VK_ESCAPE (27)
回车键： VK_RETURN (13)
TAB键： VK_TAB (9)
Caps Lock键： VK_CAPITAL (20)
Shift键： VK_SHIFT ($10)
Ctrl键： VK_CONTROL (17)
Alt键： VK_MENU (18)
空格键： VK_SPACE ($20/32)
退格键： VK_BACK (8)
左徽标键： VK_LWIN (91)
右徽标键： VK_LWIN (92)
鼠标右键快捷键：VK_APPS (93)
------------------------------------
Insert键： VK_INSERT (45)
Home键： VK_HOME (36)
Page Up： VK_PRIOR (33)
PageDown： VK_NEXT (34)
End键： VK_END (35)
Delete键： VK_DELETE (46)

方向键(←)： VK_LEFT (37)
方向键(↑)： VK_UP (38)
方向键(→)： VK_RIGHT (39)
方向键(↓)： VK_DOWN (40)

-----------------------------------
F1键： VK_F1 (112)
F2键： VK_F2 (113)
F3键： VK_F3 (114)
F4键： VK_F4 (115)
F5键： VK_F5 (116)
F6键： VK_F6 (117)
F7键： VK_F7 (118)
F8键： VK_F8 (119)
F9键： VK_F9 (120)
F10键： VK_F10 (121)
F11键： VK_F11 (122)
F12键： VK_F12 (123)

---------------------------------

Num Lock键： VK_NUMLOCK (144)
小键盘0： VK_NUMPAD0 (96)
小键盘1： VK_NUMPAD0 (97)
小键盘2： VK_NUMPAD0 (98)
小键盘3： VK_NUMPAD0 (99)
小键盘4： VK_NUMPAD0 (100)
小键盘5： VK_NUMPAD0 (101)
小键盘6： VK_NUMPAD0 (102)
小键盘7： VK_NUMPAD0 (103)
小键盘8： VK_NUMPAD0 (104)
小键盘9： VK_NUMPAD0 (105)
小键盘.： VK_DECIMAL (110)
小键盘*： VK_MULTIPLY (106)
小键盘+： VK_MULTIPLY (107)
小键盘-： VK_SUBTRACT (109)
小键盘/： VK_DIVIDE (111)
Pause Break键： VK_PAUSE (19)
Scroll Lock键： VK_SCROLL (145)

A 至 Z 键与 A – Z 字母的 ASCII 码相同：
值 描述
65 A 键
66 B 键
67 C 键
68 D 键
69 E 键
70 F 键
71 G 键
72 H 键
73 I 键
74 J 键
75 K 键
76 L 键
77 M 键
78 N 键
79 O 键
80 P 键
81 Q 键
82 R 键
83 S 键
84 T 键
85 U 键
86 V 键
87 W 键
88 X 键
89 Y 键
90 Z 键
------------------------------------------
0 至 9 键与数字 0 – 9 的 ASCII 码相同：
值 描述
48 0 键
49 1 键
50 2 键
51 3 键
52 4 键
53 5 键
54 6 键
55 7 键
56 8 键
57 9 键
————————————————
```





