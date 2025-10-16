# Tertis-Game-using-TKinter-Random

import tkinter as tk
import tkinter.simpledialog as simpledialog
import random

#Colors
colors = ["#000000","#7825B3", "#64B3B3", "#502216","#508616","#B42216", "#B4227A"]

class Figure:
    figures = [
        [[1, 5, 9, 13], [4, 5, 6, 7]],           
        [[4, 5, 9, 10], [2, 6, 5, 9]],            
        [[6, 7, 9, 10], [1, 5, 6, 10]],           
        [[1, 2, 5, 9], [0, 4, 5, 6], [1, 5, 9, 8], [4, 5, 6, 10]],  
        [[1, 2, 6, 10], [5, 6, 7, 9], [2, 6, 10, 11], [3, 5, 6, 7]],
        [[1, 4, 5, 6], [1, 4, 5, 9], [4, 5, 6, 9], [1, 5, 6, 9]],  
        [[1, 2, 5, 6]]                                          
    ]
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.type = random.randint(0, len(self.figures) - 1)
        self.color = random.randint(1, len(colors) - 1)
        self.rotation = 0
    def image(self):
        return self.figures[self.type][self.rotation]
    def rotate(self):
        self.rotation = (self.rotation + 1) % len(self.figures[self.type])

class Tetris:
    def __init__(self, height, width):
        self.level = 2
        self.score = 0
        self.high_score = 0  # Track high score
        self.state = "init"  # <- 'init' for welcome, 'start' for playing, 'gameover'
        self.height = height
        self.width = width
        self.field = [[0 for _ in range(width)] for _ in range(height)]
        self.figure = None
    def new_figure(self):
        self.figure = Figure(3, 0)
    def intersects(self):
        for i in range(4):
            for j in range(4):
                if i * 4 + j in self.figure.image():
                    if i + self.figure.y > self.height - 1 or \
                       j + self.figure.x > self.width - 1 or \
                       j + self.figure.x < 0 or \
                       self.field[i + self.figure.y][j + self.figure.x] > 0:
                        return True
        return False
    def freeze(self):
        for i in range(4):
            for j in range(4):
                if i * 4 + j in self.figure.image():
                    self.field[i + self.figure.y][j + self.figure.x] = self.figure.color
        self.break_lines()
        self.new_figure()
        if self.intersects():
            self.state = "gameover"
            self.update_high_score()
    def break_lines(self):
        lines = 0
        for i in range(self.height - 1, -1, -1):
            if 0 not in self.field[i]:
                del self.field[i]
                self.field.insert(0, [0 for _ in range(self.width)])
                lines += 1
        self.score += lines ** 2
    def go_down(self):
        if self.state == "start":
            self.figure.y += 1
            if self.intersects():
                self.figure.y -= 1
                self.freeze()
    def go_side(self, dx):
        old_x = self.figure.x
        self.figure.x += dx
        if self.intersects():
            self.figure.x = old_x
    def go_space(self):
        while not self.intersects():
            self.figure.y += 1
        self.figure.y -= 1
        self.freeze()
    def rotate(self):
        old_rotation = self.figure.rotation
        self.figure.rotate()
        if self.intersects():
            self.figure.rotation = old_rotation
    def update_high_score(self):
        if self.score > self.high_score:
            self.high_score = self.score

class TetrisApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Tetris - Tkinter Edition")

        # Field dimensions
        self.rows, self.cols = 20, 10

        # Layout frames
        self.main_frame = tk.Frame(root, bg="white")
        self.main_frame.pack(fill="both", expand=True)

        self.canvas = tk.Canvas(self.main_frame, bg="#F7F7F7")
        self.canvas.grid(row=0, column=0, rowspan=6, sticky="nsew")
        self.controls_frame = tk.Frame(self.main_frame, bg="#E6E6FA", bd=2, relief="groove")
        self.controls_frame.grid(row=0, column=1, sticky="n", padx=10, pady=10)

        self.main_frame.columnconfigure(0, weight=1)
        self.main_frame.rowconfigure(0, weight=1)

        # Player control variables
        self.num_players = 0  # Number of players (to be set dynamically)
        self.current_player = 0
        self.player_scores = []

        self.game = Tetris(self.rows, self.cols)
        self.delay = 500  # ms
        self.game_paused = False

        # Labels
        self.player_label = tk.Label(self.controls_frame, text=f"Player: 1", font=("Calibri", 14), bg="#E6E6FA")
        self.player_label.pack(pady=2)

        self.score_label = tk.Label(self.controls_frame, text="Score: 0", font=("Calibri", 16), bg="#E6E6FA")
        self.score_label.pack(pady=5)

        self.high_score_label = tk.Label(self.controls_frame, text="High Score: 0", font=("Calibri", 16, "bold"), bg="#E6E6FA", fg="#7825B3")
        self.high_score_label.pack(pady=5)

        # Control buttons
        self.start_button = tk.Button(self.controls_frame, text="Start", command=self.start_game, width=12, font=("Calibri", 12))
        self.start_button.pack(pady=6)
        self.pause_button = tk.Button(self.controls_frame, text="Pause", command=self.pause_game, width=12, font=("Calibri", 12), state="disabled")
        self.pause_button.pack(pady=6)
        self.resume_button = tk.Button(self.controls_frame, text="Resume", command=self.resume_game, width=12, font=("Calibri", 12), state="disabled")
        self.resume_button.pack(pady=6)

        # Keys
        self.root.bind("<KeyPress>", self.key_down)
        self.root.bind("<KeyRelease>", self.key_up)
        self.canvas.bind("<Configure>", self.on_resize)

        # Default canvas size
        self.canvas_width = 400
        self.canvas_height = 500
        self.zoom = 20
        self.x_offset = 50
        self.y_offset = 30

        self.show_welcome()

    def on_resize(self, event=None):
        # Handles dynamic changes to the canvas size
        if event is not None:
            self.canvas_width = event.width
            self.canvas_height = event.height

        self.zoom = min(
            (self.canvas_width - 2 * self.x_offset) // self.cols,
            (self.canvas_height - 2 * self.y_offset) // self.rows
        )
        field_w = self.cols * self.zoom
        field_h = self.rows * self.zoom
        self.x_offset = max((self.canvas_width - field_w) // 2, 0)
        self.y_offset = max((self.canvas_height - field_h) // 2, 0)
        self.redraw()

    def show_welcome(self):
        self.canvas.delete("all")
        self.game.state = "init"
        self.start_button.config(state="normal")
        self.pause_button.config(state="disabled")
        self.resume_button.config(state="disabled")
        cx = self.canvas_width // 2
        cy = self.canvas_height // 2
        self.canvas.create_text(cx, cy - 80, text="WELCOME TO TETRIS!", font=("Calibri", 32, "bold"), fill="#7825B3")
        self.canvas.create_text(cx, cy, text="Click Start to Begin", font=("Calibri", 20), fill="#508616")
        self.canvas.create_text(cx, cy + 60, text="Use Arrow keys to Move\n↑ to Rotate, ↓ to Soft Drop\nSpace to Hard Drop.",
                                font=("Calibri", 15), fill="#B4227A", justify="center")
        # Reset variables for a new game session
        self.num_players = 0
        self.current_player = 0
        self.player_scores = []

    def ask_num_players(self):
        # Pop-up dialog to ask number of players from 1 to 4
        num = simpledialog.askinteger("Number of Players", "Enter number of players (1-4):", minvalue=1, maxvalue=4)
        if num:
            self.num_players = num
        else:
            self.num_players = 0

    def start_game(self):
        # If number of players not set, ask for it
        if self.num_players == 0:
            self.ask_num_players()
            if self.num_players == 0:
                return  # User cancelled or invalid input

        # Initialize scores for players if not already done
        if len(self.player_scores) < self.num_players:
            self.player_scores = [0] * self.num_players

        # Reset or start game for the current player
        self.game = Tetris(self.rows, self.cols)
        self.game.state = "start"
        self.game_paused = False
        self.start_button.config(state="disabled")
        self.pause_button.config(state="normal")
        self.resume_button.config(state="disabled")
        self.score_label.config(text="Score: 0")
        self.high_score_label.config(text=f"High Score: {self.game.high_score}")
        self.player_label.config(text=f"Player: {self.current_player + 1}")

        # --- CRUCIAL FIX: force recalculation for every new player!
        self.on_resize(
            type("Event", (), {
                "width": self.canvas.winfo_width(),
                "height": self.canvas.winfo_height()
            })()
        )

        self.update_game()

    def pause_game(self):
        if self.game.state == "start":
            self.game_paused = True
            self.pause_button.config(state="disabled")
            self.resume_button.config(state="normal")

    def resume_game(self):
        if self.game.state == "start" and self.game_paused:
            self.game_paused = False
            self.pause_button.config(state="normal")
            self.resume_button.config(state="disabled")
            self.update_game()

    def update_game(self):
        if self.game.state == "gameover":
            # Save current player's score
            self.player_scores[self.current_player] = self.game.score

            # Move to next player or end game
            self.current_player += 1
            if self.current_player >= self.num_players:
                self.show_winner()
            else:
                # Prepare for next player's turn
                self.start_button.config(state="normal")
                self.pause_button.config(state="disabled")
                self.resume_button.config(state="disabled")
                self.score_label.config(text=f"Player {self.current_player} finished. Click Start for Player {self.current_player + 1}")
                self.player_label.config(text=f"Player: {self.current_player + 1}")
            return

        if self.game.figure is None:
            self.game.new_figure()
        if not self.game_paused:
            self.game.go_down()
            self.redraw()
            self.score_label.config(text=f"Score: {self.game.score}")
            self.high_score_label.config(text=f"High Score: {self.game.high_score}")
        if self.game.state != "gameover" and not self.game_paused:
            self.root.after(self.delay, self.update_game)

    def redraw(self):
        if self.game.state == "gameover":
            self.draw_gameover()
        elif self.game.state == "init":
            self.show_welcome()
        else:
            self.draw()

    def draw(self):
        self.canvas.delete("all")
        for i in range(self.game.height):
            for j in range(self.game.width):
                color = colors[self.game.field[i][j]]
                x = self.x_offset + j * self.zoom
                y = self.y_offset + i * self.zoom
                self.canvas.create_rectangle(x, y, x + self.zoom, y + self.zoom,
                                             outline="#E6E6FA", fill=color)
        if self.game.figure:
            for i in range(4):
                for j in range(4):
                    p = i * 4 + j
                    if p in self.game.figure.image():
                        color = colors[self.game.figure.color]
                        x = self.x_offset + (j + self.game.figure.x) * self.zoom
                        y = self.y_offset + (i + self.game.figure.y) * self.zoom
                        self.canvas.create_rectangle(x, y, x + self.zoom, y + self.zoom,
                                                     outline="black", fill=color)
        self.canvas.create_text(self.x_offset // 2 + 20, 20, text=f"Score: {self.game.score}", font=("Calibri", 16, "bold"), fill="#502216")

    def draw_gameover(self):
        self.draw()
        cx = self.canvas_width // 2
        cy = self.canvas_height // 2
        self.canvas.create_text(cx, cy - 40, text="GAME OVER", fill="orange", font=("Calibri", 40, "bold"))
        self.canvas.create_text(cx, cy + 10, text=f"High Score: {self.game.high_score}", fill="gold", font=("Calibri", 22))
        self.canvas.create_text(cx, cy + 50, text="Click Start to Continue", fill="#7825B3", font=("Calibri", 18))

    def show_winner(self):
        self.canvas.delete("all")
        max_score = max(self.player_scores)
        winners = [i + 1 for i, score in enumerate(self.player_scores) if score == max_score]

        message = "Game Over!\n\n"
        for i, score in enumerate(self.player_scores, start=1):
            message += f"Player {i}: {score}\n"
        if len(winners) == 1:
            message += f"\nWinner: Player {winners[0]}"
        else:
            message += f"\nIt's a tie between players {', '.join(map(str, winners))}!"

        self.canvas.create_text(self.canvas_width // 2, self.canvas_height // 2,
                                text=message, fill="purple", font=("Calibri", 24), justify="center")

        self.start_button.config(state="disabled")
        self.pause_button.config(state="disabled")
        self.resume_button.config(state="disabled")

    def key_down(self, event):
        if self.game.state != "start" or self.game_paused:
            return
        if event.keysym == "Left":
            self.game.go_side(-1)
        elif event.keysym == "Right":
            self.game.go_side(1)
        elif event.keysym == "Up":
            self.game.rotate()
        elif event.keysym == "Down":
            self.game.go_down()
        elif event.keysym == "space":
            self.game.go_space()
        self.redraw()
        if event.keysym == "Escape":
            self.show_welcome()

    def key_up(self, event):
        pass

root = tk.Tk()
root.minsize(480, 560)
app = TetrisApp(root)
root.mainloop()
