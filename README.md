import tkinter as tk
from tkinter import messagebox
from datetime import datetime

class LogStore:
    def __init__(self): self.logs = []
    def add(self, u, a): self.logs.append((u, a, datetime.now()))
    def all(self): return self.logs

class Painter:
    def __init__(self, c): self.c, self.sz, self.col = c, 5, "black"
    def set_sz(self, s): self.sz = s
    def set_col(self, c): self.col = c
    def draw(self, x, y): r = self.sz; self.c.create_oval(x-r, y-r, x+r, y+r, fill=self.col, outline=self.col)
    def clear(self): self.c.delete("all")

class App(tk.Tk):
    COLORS = [("Черный","black"),("Красный","red"),("Зеленый","green"),("Синий","blue"),("Белый","white"),
              ("Оранжевый","orange"),("Желтый","yellow"),("Розовый","pink"),("Фиолетовый","violet")]
    SIZES = [5,10,15,20,30,50,100]

    def __init__(self):
        super().__init__()
        self.title("Paint OOP"); self.geometry("900x700")
        self.store = LogStore(); self.painter = None
        self._build(); self.protocol("WM_DELETE_WINDOW", self.on_exit)

    def _build(self):
        f = tk.Frame(self, bg="#333", pady=10); f.pack(fill=tk.X)
        tk.Label(f, text="Цвета:", bg="#333", fg="white", font=("Arial",10,"bold")).pack(side=tk.LEFT, padx=10)
        for n, c in self.COLORS:
            btn = tk.Button(f, text=n, width=8, bg=c, fg="white" if c in ("black","blue","violet") else "black",
                            font=("Arial",9,"bold"), command=lambda col=c: self.set_color(col))
            btn.pack(side=tk.LEFT, padx=2, pady=2)
        tk.Label(f, text=" Размер:", bg="#333", fg="white", font=("Arial",10,"bold")).pack(side=tk.LEFT, padx=20)
        for s in self.SIZES:
            btn = tk.Button(f, text=f"{s}px", width=5, bg="#f0f0f0", relief=tk.RAISED,
                            font=("Arial",9,"bold"), command=lambda sz=s: self.set_size(sz))
            btn.pack(side=tk.LEFT, padx=2)
        tk.Button(f, text="Очистить", width=12, bg="#ff6b6b", fg="white",
                  font=("Arial",10,"bold"), command=self.clear).pack(side=tk.LEFT, padx=20)
        tk.Button(f, text="Отчет", width=12, bg="#4ecdc4", fg="white",
                  font=("Arial",10,"bold"), command=self.save).pack(side=tk.LEFT)
        self.canvas = tk.Canvas(self, bg="white", cursor="cross", highlightthickness=2, highlightbackground="#ccc")
        self.canvas.pack(fill=tk.BOTH, expand=1, padx=10, pady=10)
        self.painter = Painter(self.canvas)
        self.canvas.bind("<B1-Motion>", self.draw)
        lf = tk.LabelFrame(self, text="Журнал", font=("Arial",10,"bold")); lf.pack(fill=tk.X, padx=10, pady=5)
        self.log = tk.Text(lf, height=4, state="disabled", bg="#e8f4f8", font=("Consolas",9)); self.log.pack(fill=tk.X)
        self.status = tk.StringVar(value=f"Цвет: {self.painter.col} | Размер: {self.painter.sz}px")
        tk.Label(self, textvariable=self.status, bd=1, relief=tk.SUNKEN, anchor=tk.W).pack(fill=tk.X)
        self.store.add("system", "App Started"); self.append_log("Старт")

    def append_log(self, msg):
        self.log.config(state="normal"); self.log.insert(tk.END, msg + "\n"); self.log.see(tk.END); self.log.config(state="disabled")

    def set_color(self, c):
        self.painter.set_col(c); self.status.set(f"Цвет: {c} | Размер: {self.painter.sz}px")
        self.store.add("user", f"Color {c}"); self.append_log(f"Цвет: {c}")

    def set_size(self, s):
        self.painter.set_sz(s); self.status.set(f"Цвет: {self.painter.col} | Размер: {s}px")
        self.store.add("user", f"Size {s}"); self.append_log(f"Размер: {s}px")

    def draw(self, e): self.painter.draw(e.x, e.y); self.store.add("user", "Draw")
    def clear(self): self.painter.clear(); self.store.add("user", "Clear"); self.append_log("Очистка")

    def save(self):
        fn = "diagnostic.md"
        try:
            now = datetime.now()
            with open(fn, "w", encoding="utf-8") as f:
                f.write("| login | time_in | time_out |\n|-------|---------|----------|\n")
                for u, a, t in self.store.all():
                    f.write(f"| {u} | {t.strftime('%H:%M')} | {now.strftime('%H:%M')} |\n")
            self.append_log(f"Отчет: {fn}"); messagebox.showinfo("OK", f"Отчет: {fn}")
        except Exception as e: messagebox.showerror("Err", str(e))

    def on_exit(self):
        if messagebox.askyesno("Выход", "Выйти?"): self.save(); self.destroy()

if __name__ == "__main__": App().mainloop()
