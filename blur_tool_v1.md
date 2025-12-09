# Blur Tool – Full Beginner Setup Guide

This guide explains everything from **installing Python** to **running your blur tool**, **auto‑startup**, and **using hotkeys**.
Perfect for beginners.

---

## ✅ 1. Install the Correct Python Version (Python 3.12)
**Do NOT use Python 3.14** because NumPy & OpenCV do not support it yet.

### Step 1 — Download Python 3.12
Download from:

```
https://www.python.org/ftp/python/3.12.7/python-3.12.7-amd64.exe
```

### Step 2 — Install
- Check **"Add Python to PATH"** ✔️
- Choose **Install Now**

### Step 3 — Confirm Installation
Open Command Prompt:
```
python --version
```
Should show:
```
Python 3.12.x
```

---

## ✅ 2. Install Required Packages
Open CMD and run:
```
pip install pillow numpy opencv-python keyboard pywin32
```

All packages will install successfully on Python 3.12.

---

## ✅ 3. Prepare Your Folder
Create a folder:
```
D:\My Code
```

Save your final updated file as:
```
blur_tool.py
```

---

## ✅ 4. How to Run the Tool Manually
To test the tool:
```
cd /d "D:\My Code"
python blur_tool.py
```

---

## ✅ 5. Hotkeys (Latest)
| Hotkey     | Action |
|------------|--------|
| **Shift + H** | Turn tool ON/OFF (background mode). |
| **Shift + A** | Enable / Disable drawing mode. |
| **Shift + X** | Reset all blur + rectangles + numbers. |
| **Shift + B** | Remove blur and restore screen to normal. |
| **Left Mouse** | Draw a red rectangle (blur area). |
| **Esc** | Close tool completely. |

---

## ✅ 6. Add Tool to Windows Startup
This ensures tool runs automatically after restart.

### Step 1 — Create a startup script
Create a file:
```
D:\My Code\start_blur_tool.bat
```
Add this inside:
```
@echo off
cd /d "D:\My Code"
start python blur_tool.py
```
 Save it.

### Step 2 — Open Startup Folder
Press:
```
Win + R
```
Type:
```
shell:startup
```
Press Enter.

### Step 3 — Add to Startup
Copy this file into the folder:
```
start_blur_tool.bat
```

Now your blur tool will auto‑start every time you boot your laptop.

---

## ✅ 7. How the Tool Works After Auto‑Start
When Windows starts:
- Tool runs **silently in background**.
- You can turn it ON/OFF anytime using:

```
Shift + H
```
- You do NOT need CMD again.

---

## ✅ 8. If You Update the Script Later
Just replace the file:
```
D:\My Code\blur_tool.py
```
Auto‑startup will still work.

---

## 🎉 You Are Fully Set Up!
You now have:
- Working blur tool
- Auto‑startup enabled
- Updated hotkeys
- Stable Python version
- Full step‑by‑step beginner guide

If you want, I can also generate:
- Installer EXE
- System tray icon version
- Full GUI version

Just tell me! 🚀


## Full Python Script (blur_tool.py)
```python
import tkinter as tk
import win32gui, win32con
import ctypes
import numpy as np
import cv2
from PIL import ImageGrab, ImageTk
import keyboard
import threading
import time

class BlurTool:
    def __init__(self):
        self.root = tk.Tk()
        self.root.attributes("-fullscreen", True)
        self.root.attributes("-topmost", True)
        self.root.attributes("-alpha", 0.3)
        self.root.config(bg="black")
        self.root.withdraw() 

        self.active = False
        self.can_draw = True
        self.rectangles = []
        self.labels = []
        self.drawing = False
        self.start_x = 0
        self.start_y = 0
        self.rect = None
        self.label_counter = 1

        self.canvas = tk.Canvas(self.root, bg="", highlightthickness=0)
        self.canvas.pack(fill="both", expand=True)

        self.canvas.bind("<ButtonPress-1>", self.on_start)
        self.canvas.bind("<B1-Motion>", self.on_draw)
        self.canvas.bind("<ButtonRelease-1>", self.on_release)

        keyboard.add_hotkey("shift+h", self.toggle_tool)
        keyboard.add_hotkey("shift+a", self.toggle_drawing)
        keyboard.add_hotkey("shift+x", self.reset_all)
        keyboard.add_hotkey("shift+b", self.clear_blur)
        keyboard.add_hotkey("esc", self.close_tool)

        threading.Thread(target=self.root.mainloop, daemon=True).start()

    def toggle_tool(self):
        if self.active:
            self.root.withdraw()
        else:
            self.root.deiconify()
        self.active = not self.active

    def toggle_drawing(self):
        self.can_draw = not self.can_draw

    def on_start(self, event):
        if not self.can_draw:
            return
        self.drawing = True
        self.start_x = event.x
        self.start_y = event.y
        self.rect = self.canvas.create_rectangle(event.x, event.y, event.x, event.y, outline="red", width=3)

    def on_draw(self, event):
        if self.drawing:
            self.canvas.coords(self.rect, self.start_x, self.start_y, event.x, event.y)

    def on_release(self, event):
        if not self.drawing:
            return
        self.drawing = False
        x1, y1, x2, y2 = self.canvas.coords(self.rect)
        self.apply_blur(int(x1), int(y1), int(x2), int(y2))
        label = self.canvas.create_text(x1+10, y1+10, text=str(self.label_counter), fill="red", anchor="nw", font=("Arial", 18, "bold"))
        self.label_counter += 1
        self.rectangles.append(self.rect)
        self.labels.append(label)
        self.rect = None

    def apply_blur(self, x1, y1, x2, y2):
        img = np.array(ImageGrab.grab())
        sub = img[y1:y2, x1:x2]
        blur = cv2.GaussianBlur(sub, (51, 51), 0)
        img[y1:y2, x1:x2] = blur
        cv2.imshow("Blurred", img)
        cv2.waitKey(1)

    def reset_all(self):
        for r in self.rectangles:
            self.canvas.delete(r)
        for l in self.labels:
            self.canvas.delete(l)
        self.rectangles.clear()
        self.labels.clear()
        self.label_counter = 1
        cv2.destroyAllWindows()

    def clear_blur(self):
        cv2.destroyAllWindows()

    def close_tool(self):
        self.reset_all()
        cv2.destroyAllWindows()
        self.root.destroy()
        raise SystemExit

if __name__ == "__main__":
    BlurTool()
```

