# roguelike_minesweeper.py

import tkinter as tk
from tkinter import messagebox, Toplevel
import random


class RoguelikeMinesweeper:
    def __init__(self):
        self.level = 1
        self.player_hp = 5
        self.player_max_hp = 5
        self.gold = 0
        self.exp = 0
        self.exp_to_next = 3
        self.flags = 5  # 初始标记次数

        # 猜拳统计
        self.win_streak = 0
        self.total_fights = 0

        self.grid = []
        self.revealed = []
        self.flagged = []
        self.monsters = set()
        self.treasures = set()
        self.mines = set()
        self.player_pos = [0, 0]

        self.rows = 0
        self.cols = 0
        self.mine_count = 0
        self.game_over = False
        self.win = False
        self.mines_placed = False
        self.fighting = False  # 是否在战斗中

        self.root = tk.Tk()
        self.root.title(f"⚔️ Roguelike扫雷 - 第{self.level}层")
        self.root.resizable(False, False)

        # 键盘绑定
        self.root.bind('<Up>', lambda e: self.move_player(-1, 0))
        self.root.bind('<Down>', lambda e: self.move_player(1, 0))
        self.root.bind('<Left>', lambda e: self.move_player(0, -1))
        self.root.bind('<Right>', lambda e: self.move_player(0, 1))
        self.root.bind('<space>', lambda e: self.reveal_current())
        self.root.bind('<Return>', lambda e: self.reveal_current())
        self.root.bind('<f>', lambda e: self.toggle_flag())
        self.root.bind('<F>', lambda e: self.toggle_flag())
        self.root.bind('<r>', lambda e: self.reset_game())
        self.root.bind('<R>', lambda e: self.reset_game())
        self.root.bind('<Escape>', lambda e: self.root.quit())

        self.setup_ui()
        self.init_game()

    def setup_ui(self):
        info_frame = tk.Frame(self.root)
        info_frame.pack(pady=5)

        self.hp_label = tk.Label(info_frame, text=f"❤️ {self.player_hp}/{self.player_max_hp}",
                                 font=('Arial', 12, 'bold'), fg='red')
        self.hp_label.pack(side=tk.LEFT, padx=8)

        self.gold_label = tk.Label(info_frame, text=f"💰 {self.gold}",
                                   font=('Arial', 12, 'bold'), fg='gold')
        self.gold_label.pack(side=tk.LEFT, padx=8)

        self.level_label = tk.Label(info_frame, text=f"🏰 第{self.level}层",
                                    font=('Arial', 12, 'bold'), fg='purple')
        self.level_label.pack(side=tk.LEFT, padx=8)

        self.flags_label = tk.Label(info_frame, text=f"🚩 {self.flags}",
                                    font=('Arial', 12, 'bold'), fg='blue')
        self.flags_label.pack(side=tk.LEFT, padx=8)

        self.threat_label = tk.Label(info_frame, text=f"💣 地雷: 0 | 👾 怪物: 0",
                                     font=('Arial', 12, 'bold'), fg='red')
        self.threat_label.pack(side=tk.LEFT, padx=8)

        # 操作提示
        hint_frame = tk.Frame(self.root)
        hint_frame.pack(pady=2)

        self.hint_label = tk.Label(hint_frame,
                                   text="💡 方向键移动 | 空格翻开 | F标记地雷 | R重置 | ESC退出",
                                   font=('Arial', 9), fg='gray')
        self.hint_label.pack()

        self.status_label = tk.Label(hint_frame, text="📍 准备开始",
                                     font=('Arial', 10), fg='blue')
        self.status_label.pack()

        # 控制按钮
        control_frame = tk.Frame(self.root)
        control_frame.pack(pady=5)

        tk.Button(control_frame, text="🔄 重新开始 (R)",
                  command=self.reset_game).pack(side=tk.LEFT, padx=3)
        tk.Button(control_frame, text="⬆️", width=3,
                  command=lambda: self.move_player(-1, 0)).pack(side=tk.LEFT, padx=1)
        tk.Button(control_frame, text="⬇️", width=3,
                  command=lambda: self.move_player(1, 0)).pack(side=tk.LEFT, padx=1)
        tk.Button(control_frame, text="⬅️", width=3,
                  command=lambda: self.move_player(0, -1)).pack(side=tk.LEFT, padx=1)
        tk.Button(control_frame, text="➡️", width=3,
                  command=lambda: self.move_player(0, 1)).pack(side=tk.LEFT, padx=1)
        tk.Button(control_frame, text="🔍 翻开 (空格)", width=10,
                  command=self.reveal_current).pack(side=tk.LEFT, padx=3)
        tk.Button(control_frame, text="🚩 标记 (F)", width=8,
                  command=self.toggle_flag).pack(side=tk.LEFT, padx=3)

        self.board_frame = tk.Frame(self.root)
        self.board_frame.pack(padx=10, pady=5)

        self.buttons = []
        self.cell_size = 30

    def init_game(self):
        """初始化游戏"""
        self.rows = 10 + self.level * 2
        self.cols = 10 + self.level * 2
        self.mine_count = 8 + self.level * 3

        self.grid = [[0 for _ in range(self.cols)] for _ in range(self.rows)]
        self.revealed = [[False for _ in range(self.cols)] for _ in range(self.rows)]
        self.flagged = [[False for _ in range(self.cols)] for _ in range(self.rows)]
        self.mines = set()
        self.monsters = set()
        self.treasures = set()
        self.mines_placed = False
        self.game_over = False
        self.win = False

        self.flags = self.mine_count + 3

        for widget in self.board_frame.winfo_children():
            widget.destroy()
        self.buttons = []

        for r in range(self.rows):
            row_buttons = []
            for c in range(self.cols):
                btn = tk.Button(
                    self.board_frame,
                    width=2,
                    height=1,
                    font=('Arial', 10),
                    relief=tk.RAISED,
                    bg='#c0c0c0'
                )
                btn.grid(row=r, column=c, padx=1, pady=1)
                row_buttons.append(btn)
            self.buttons.append(row_buttons)

        self.player_pos = [self.rows // 2, self.cols // 2]
        self.update_ui()
        self.status_label.config(text=f"📍 按 空格/回车 翻开第一格 (标记次数: {self.flags})")

    def place_features(self, safe_r, safe_c):
        """放置地雷、怪物和宝藏"""
        safe_zone = set()
        for dr in range(-2, 3):
            for dc in range(-2, 3):
                nr, nc = safe_r + dr, safe_c + dc
                if 0 <= nr < self.rows and 0 <= nc < self.cols:
                    safe_zone.add((nr, nc))

        positions = [(r, c) for r in range(self.rows) for c in range(self.cols)
                     if (r, c) not in safe_zone]
        random.shuffle(positions)

        if len(positions) < self.mine_count + self.mine_count // 3 + self.mine_count // 4:
            safe_zone = set()
            for dr in range(-1, 2):
                for dc in range(-1, 2):
                    nr, nc = safe_r + dr, safe_c + dc
                    if 0 <= nr < self.rows and 0 <= nc < self.cols:
                        safe_zone.add((nr, nc))
            positions = [(r, c) for r in range(self.rows) for c in range(self.cols)
                         if (r, c) not in safe_zone]
            random.shuffle(positions)

        idx = 0

        # 放置地雷
        for _ in range(min(self.mine_count, len(positions) - idx)):
            r, c = positions[idx]
            self.mines.add((r, c))
            self.grid[r][c] = -1
            for dr in [-1, 0, 1]:
                for dc in [-1, 0, 1]:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < self.rows and 0 <= nc < self.cols:
                        if self.grid[nr][nc] >= 0:
                            self.grid[nr][nc] += 1
            idx += 1

        # 放置怪物
        monster_count = min(self.mine_count // 3, (len(positions) - idx) // 2)
        for _ in range(monster_count):
            r, c = positions[idx]
            self.monsters.add((r, c))
            self.grid[r][c] = -2
            idx += 1

        # 放置宝藏
        treasure_count = min(self.mine_count // 4, len(positions) - idx)
        for _ in range(treasure_count):
            r, c = positions[idx]
            self.treasures.add((r, c))
            self.grid[r][c] = -3
            idx += 1

        self.mines_placed = True
        self.update_ui()
        self.status_label.config(text=f"🗺️ 地雷: {len(self.mines)} | 标记: {self.flags} | 怪物: {len(self.monsters)}")

    def toggle_flag(self):
        """标记/取消标记当前格子的地雷 - 手操"""
        if self.game_over or self.fighting:
            return

        r, c = self.player_pos

        if self.revealed[r][c]:
            self.status_label.config(text="⚠️ 已翻开的格子不能标记")
            return

        # 检查当前位置是否是地雷
        is_mine = (r, c) in self.mines

        if self.flagged[r][c]:
            self.flagged[r][c] = False
            self.flags += 1
            self.buttons[r][c].config(text='', bg='#c0c0c0')
            self.status_label.config(text=f"↩️ 取消标记，剩余 {self.flags} 次")
        else:
            if self.flags <= 0:
                self.status_label.config(text="⚠️ 标记次数用完了！")
                return

            self.flagged[r][c] = True
            self.flags -= 1
            self.buttons[r][c].config(text='🚩', bg='#ffcccc')

            if is_mine:
                # ✅ 正确标记地雷
                self.gold += 3
                self.exp += 1
                self.mines.remove((r, c))
                self.grid[r][c] = 0
                self.status_label.config(text=f"✅ 地雷标记正确！+3金币，剩余 {self.flags} 次")

                if self.exp >= self.exp_to_next:
                    self.exp_up()
                self.check_win()
            else:
                # ❌ 错误标记惩罚 - 扣1血并失去标记
                self.player_hp -= 1
                self.flagged[r][c] = False  # 取消标记
                self.buttons[r][c].config(text='❌', bg='red')
                self.status_label.config(text=f"❌ 错误标记！这里没有地雷！-1 HP")

                # 1秒后恢复按钮外观
                self.root.after(1000, lambda: self.buttons[r][c].config(text='', bg='#c0c0c0'))

                if self.player_hp <= 0:
                    self.game_over = True
                    messagebox.showinfo("💀 游戏结束", f"错误标记导致死亡！\n到达第{self.level}层\n💰 金币: {self.gold}")

        self.update_ui()

    def exp_up(self):
        """经验升级 - 奖励标记次数"""
        self.exp = 0
        self.exp_to_next = min(10, self.exp_to_next + 2)
        self.player_max_hp += 1
        self.player_hp = min(self.player_hp + 1, self.player_max_hp)
        self.flags += 2

        messagebox.showinfo("⭐ 升级!",
                            f"等级提升！\n❤️ HP: {self.player_hp}/{self.player_max_hp}\n🚩 标记 +2")
        self.update_ui()

    def move_player(self, dr, dc):
        """移动玩家 - 触碰触发怪物和宝藏"""
        if self.game_over or self.fighting:
            return

        r, c = self.player_pos
        nr, nc = r + dr, c + dc

        if not (0 <= nr < self.rows and 0 <= nc < self.cols):
            self.status_label.config(text="⚠️ 超出边界！")
            return

        self.player_pos = [nr, nc]

        # ⭐ 检查是否移动到怪物格子（触碰触发）
        if (nr, nc) in self.monsters:
            # 先显示怪物
            self.revealed[nr][nc] = True
            self.buttons[nr][nc].config(text='👾', bg='orange', relief=tk.SUNKEN)
            self.status_label.config(text="👾 遭遇怪物！准备战斗！")
            self.start_fight(nr, nc)
            return

        # ⭐ 检查是否移动到宝藏格子（触碰触发）
        if (nr, nc) in self.treasures:
            # 先显示宝藏
            self.revealed[nr][nc] = True
            self.buttons[nr][nc].config(text='💰', bg='gold', relief=tk.SUNKEN)
            self.treasures.remove((nr, nc))
            self.grid[nr][nc] = 0
            reward = random.randint(5, 15)
            self.gold += reward
            self.exp += 1
            self.flags += 1
            self.status_label.config(text=f"💰 发现宝藏！获得 {reward} 金币！+1标记")

            if self.exp >= self.exp_to_next:
                self.exp_up()
            self.update_ui()
            return

        # 普通移动
        self.update_ui()
        self.status_label.config(text=f"📍 当前位置: ({nr}, {nc})")

    def start_fight(self, r, c):
        """开始与怪物猜拳战斗"""
        self.fighting = True
        self.status_label.config(text="⚔️ 进入战斗！选择你的出拳")

        # 创建战斗窗口
        fight_window = Toplevel(self.root)
        fight_window.title("⚔️ 猜拳战斗！")
        fight_window.geometry("300x400")
        fight_window.resizable(False, False)
        fight_window.transient(self.root)
        fight_window.grab_set()

        # 显示怪物信息
        info_label = tk.Label(fight_window, text="👾 遭遇怪物！\n选择你的出拳方式：",
                              font=('Arial', 14, 'bold'))
        info_label.pack(pady=10)

        # 显示结果区域
        result_label = tk.Label(fight_window, text="", font=('Arial', 12), height=3)
        result_label.pack(pady=5)

        # 按钮框架
        button_frame = tk.Frame(fight_window)
        button_frame.pack(pady=10)

        # 石头剪刀布按钮
        def fight_choice(player_choice):
            if fight_window.winfo_exists():
                self.fight_result(r, c, player_choice, fight_window, result_label)

        # 创建三个选择按钮（带表情符号）
        choices = [
            ("🪨 石头", "石头"),
            ("✂️ 剪刀", "剪刀"),
            ("📄 布", "布")
        ]

        for text, choice in choices:
            btn = tk.Button(button_frame, text=text, font=('Arial', 12, 'bold'),
                            width=10, height=2,
                            command=lambda c=choice: fight_choice(c))
            btn.pack(pady=3)

        # 逃跑按钮（有代价）
        def flee():
            if fight_window.winfo_exists():
                self.player_hp -= 1
                fight_window.destroy()
                self.fighting = False
                self.status_label.config(text="🏃 逃跑成功！但损失了1 HP")
                self.update_ui()
                if self.player_hp <= 0:
                    self.game_over = True
                    messagebox.showinfo("💀 游戏结束", "逃跑时被怪物杀死！")

        flee_btn = tk.Button(fight_window, text="🏃 逃跑 (-1 HP)", font=('Arial', 10),
                             bg='orange', command=flee)
        flee_btn.pack(pady=5)

    def fight_result(self, r, c, player_choice, fight_window, result_label):
        """处理战斗结果"""
        # 电脑随机选择
        choices = ["石头", "剪刀", "布"]
        computer_choice = random.choice(choices)

        # 判断胜负
        result = self.determine_winner(player_choice, computer_choice)

        # 显示战斗动画效果
        fight_window.geometry("300x450")

        # 显示双方选择
        choice_text = f"你: {player_choice}  🤖 怪物: {computer_choice}\n\n"

        if result == "win":
            choice_text += "🎉 你赢了！\n"
            # 胜利奖励
            self.monsters.remove((r, c))
            self.grid[r][c] = 0
            self.revealed[r][c] = True
            self.buttons[r][c].config(text='⚔️', bg='green', relief=tk.SUNKEN)
            self.gold += 10
            self.exp += 2
            self.flags += 2
            self.win_streak += 1
            self.total_fights += 1

            self.status_label.config(text=f"⚔️ 猜拳胜利！+10金币，+2标记")

            if self.exp >= self.exp_to_next:
                self.exp_up()
            self.check_win()

        elif result == "lose":
            choice_text += "💀 你输了！\n"
            # 失败惩罚
            self.player_hp -= 1
            self.win_streak = 0
            self.total_fights += 1

            self.status_label.config(text=f"💀 猜拳失败！-1 HP")

            if self.player_hp <= 0:
                self.game_over = True
                result_label.config(text=choice_text + "\n💀 你被怪物杀死了！", fg='red')
                self.status_label.config(text="💀 游戏结束")
                fight_window.after(1500, lambda: self.close_fight_window(fight_window))
                self.update_ui()
                return
        else:  # draw
            choice_text += "🤝 平局！再打一次！"
            self.status_label.config(text="🤝 平局！继续战斗！")
            result_label.config(text=choice_text + "\n🤝 再来一次！", fg='orange')
            # 不清除窗口，让玩家再选一次
            return

        # 显示结果并关闭窗口
        if result == "win":
            result_label.config(text=choice_text, fg='green')
        else:
            result_label.config(text=choice_text, fg='red')

        # 更新UI
        self.update_ui()

        # 延迟关闭战斗窗口
        fight_window.after(2000, lambda: self.close_fight_window(fight_window))

    def close_fight_window(self, fight_window):
        """关闭战斗窗口"""
        try:
            fight_window.destroy()
        except:
            pass
        self.fighting = False

    def determine_winner(self, player, computer):
        """判断猜拳胜负"""
        if player == computer:
            return "draw"
        elif (player == "石头" and computer == "剪刀") or \
                (player == "剪刀" and computer == "布") or \
                (player == "布" and computer == "石头"):
            return "win"
        else:
            return "lose"

    def reveal_current(self):
        """翻开当前所在格子 - 手操"""
        if self.game_over or self.fighting:
            return

        r, c = self.player_pos

        if self.revealed[r][c]:
            self.status_label.config(text="⚠️ 这个格子已经翻开了！")
            return

        if self.flagged[r][c]:
            self.status_label.config(text="⚠️ 这个格子已被标记，取消标记再翻开")
            return

        if not self.mines_placed:
            self.place_features(r, c)

        self.reveal(r, c)

    def reveal(self, r, c):
        """翻开格子 - 手操翻开"""
        if not (0 <= r < self.rows and 0 <= c < self.cols):
            return

        if self.revealed[r][c]:
            return

        if self.flagged[r][c]:
            return

        self.revealed[r][c] = True
        btn = self.buttons[r][c]
        btn.config(relief=tk.SUNKEN)

        val = self.grid[r][c]

        # 踩到地雷
        if val == -1 and (r, c) in self.mines:
            btn.config(text='💣', bg='red')
            self.player_hp -= 1
            self.mines.remove((r, c))
            self.grid[r][c] = 0
            self.update_ui()
            self.status_label.config(text=f"💥 踩到地雷！-1 HP，剩余 {len(self.mines)} 个地雷")

            if self.player_hp <= 0:
                self.game_over = True
                messagebox.showinfo("💀 游戏结束", f"你踩到地雷死了！\n到达第{self.level}层\n💰 金币: {self.gold}")
            else:
                self.check_win()
            return

        # ⭐ 怪物 - 翻开时显示为问号（统一颜色）
        if val == -2:
            btn.config(text='❓', fg='#666666', bg='#f0e6d3')
            self.status_label.config(text="❓ 发现可疑痕迹...")
            self.update_ui()
            return

        # ⭐ 宝藏 - 翻开时显示为问号（统一颜色）
        if val == -3:
            btn.config(text='❓', fg='#666666', bg='#f0e6d3')
            self.status_label.config(text="❓ 发现可疑痕迹...")
            self.update_ui()
            return

        # 数字显示
        if val > 0:
            colors = ['', 'blue', 'green', 'red', 'darkblue', 'darkred', 'teal', 'black', 'gray']
            btn.config(text=str(val), fg=colors[min(val, 8)], bg='#f5e6d3')
            self.status_label.config(text=f"🔢 数字 {val}")

        # 空白展开
        if val == 0:
            btn.config(text='', bg='#e8f0f8')
            self.status_label.config(text="⬜ 空白区域")
            queue = [(r, c)]
            visited = {(r, c)}
            while queue:
                cr, cc = queue.pop(0)
                for dr in [-1, 0, 1]:
                    for dc in [-1, 0, 1]:
                        nr, nc = cr + dr, cc + dc
                        if 0 <= nr < self.rows and 0 <= nc < self.cols:
                            if not self.revealed[nr][nc] and (nr, nc) not in visited:
                                if self.grid[nr][nc] >= 0:
                                    visited.add((nr, nc))
                                    self.reveal(nr, nc)
                                # ⭐ 展开时遇到怪物或宝藏，只显示问号（统一颜色）
                                elif self.grid[nr][nc] == -2:
                                    visited.add((nr, nc))
                                    self.revealed[nr][nc] = True
                                    self.buttons[nr][nc].config(text='❓', fg='#666666', bg='#f0e6d3', relief=tk.SUNKEN)
                                elif self.grid[nr][nc] == -3:
                                    visited.add((nr, nc))
                                    self.revealed[nr][nc] = True
                                    self.buttons[nr][nc].config(text='❓', fg='#666666', bg='#f0e6d3', relief=tk.SUNKEN)

        self.update_ui()
        self.check_win()

    def check_win(self):
        """检查是否所有地雷已标记完成"""
        if self.game_over:
            return

        # 只要地雷全部被清除即可过关（怪物不影响）
        if len(self.mines) == 0:
            self.win = True
            self.game_over = True
            self.status_label.config(text="🎉 所有地雷已清除！进入下一层！")
            self.root.after(500, self.go_to_next_level)

    def go_to_next_level(self):
        """进入下一层"""
        self.level += 1
        self.root.title(f"⚔️ Roguelike扫雷 - 第{self.level}层")

        self.flags = (self.mine_count + 3) + self.flags // 2

        self.init_game()
        self.mines_placed = False
        self.status_label.config(text=f"🏰 第{self.level}层开始！标记次数: {self.flags}")
        messagebox.showinfo("🏰 进入下一层",
                            f"🎉 地雷全部清除！\n"
                            f"欢迎来到第 {self.level} 层！\n"
                            f"❤️ HP: {self.player_hp}/{self.player_max_hp}\n"
                            f"💰 金币: {self.gold}\n"
                            f"🚩 标记: {self.flags}")
        self.update_ui()

    def update_ui(self):
        """更新UI - 区分翻开格子和当前所在格子"""
        self.hp_label.config(text=f"❤️ {self.player_hp}/{self.player_max_hp}")
        self.gold_label.config(text=f"💰 {self.gold}")
        self.level_label.config(text=f"🏰 第{self.level}层")
        self.flags_label.config(text=f"🚩 {self.flags}")
        self.threat_label.config(text=f"💣 地雷: {len(self.mines)} | 👾 怪物: {len(self.monsters)}")

        pr, pc = self.player_pos

        for r in range(self.rows):
            for c in range(self.cols):
                btn = self.buttons[r][c]
                is_player = (r == pr and c == pc)
                is_revealed = self.revealed[r][c]

                if is_revealed:
                    # 已翻开的格子
                    val = self.grid[r][c]
                    text = btn.cget('text')

                    # 特殊符号保持显示
                    if text in ['👾', '💰', '💣', '🚩', '⚔️', '❌', '❓']:
                        pass
                    elif val > 0:
                        btn.config(bg='#f5e6d3', relief=tk.SUNKEN)
                    else:
                        btn.config(bg='#e8f0f8', relief=tk.SUNKEN)

                    # 如果玩家在已翻开的格子上，用亮绿色高亮
                    if is_player:
                        btn.config(bg='#66ff66')
                else:
                    # 未翻开的格子
                    if is_player:
                        # 玩家当前位置 - 用亮绿色
                        btn.config(bg='#88ff88', relief=tk.RAISED)
                    else:
                        # 普通未翻开格子
                        btn.config(bg='#c0c0c0', relief=tk.RAISED)

    def reset_game(self):
        """重置游戏"""
        self.level = 1
        self.player_hp = 5
        self.player_max_hp = 5
        self.gold = 0
        self.exp = 0
        self.exp_to_next = 3
        self.flags = 5
        self.win_streak = 0
        self.total_fights = 0
        self.fighting = False
        self.root.title("⚔️ Roguelike扫雷 - 第1层")
        self.init_game()
        self.mines_placed = False
        self.status_label.config(text="🔄 已重置！按 空格/回车 开始")

    def run(self):
        self.root.focus_set()
        self.root.mainloop()


if __name__ == "__main__":
    print("=" * 50)
    print("⚔️ Roguelike扫雷 - 惊喜探索版")
    print("=" * 50)
    print("⌨️  键盘控制:")
    print("  ↑ ↓ ← →  移动角色")
    print("  空格/回车  翻开当前格子 (手操)")
    print("  F         标记/取消标记地雷 (手操)")
    print("  R         重新开始")
    print("  ESC       退出游戏")
    print("=" * 50)
    print("💡 惊喜机制:")
    print("  ❓ 翻开格子时，怪物和宝藏显示为问号（统一颜色）")
    print("  👾 只有移动到怪物格子才触发战斗 (触碰触发)")
    print("  💰 只有移动到宝藏格子才获得奖励 (触碰触发)")
    print("=" * 50)
    print("🎨 颜色区分:")
    print("  🟩 亮绿色 = 玩家当前位置")
    print("  🟦 浅蓝色 = 已翻开的空白格")
    print("  🟫 米色 = 已翻开的数字格")
    print("  ⬜ 灰色 = 未翻开的格子")
    print("=" * 50)
    print("💡 战斗系统:")
    print("  ⚔️ 遇到怪物 → 进入猜拳对决")
    print("  🪨 石头 > ✂️ 剪刀 > 📄 布 > 🪨 石头")
    print("  🏃 逃跑 -1 HP (惩罚)")
    print("=" * 50)
    print("💡 标记系统 (手操):")
    print("  ✅ 正确标记 → +3金币，+1经验")
    print("  ❌ 错误标记 → -1 HP（惩罚）")
    print("=" * 50)
    print("🎯 过关条件:")
    print("  💣 清除所有地雷即可进入下一层")
    print("  👾 怪物可选打或不打")
    print("=" * 50)

    game = RoguelikeMinesweeper()
    game.run()
