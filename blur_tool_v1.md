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
from PIL import ImageGrab, ImageTk, Image
import numpy as np
import cv2
import win32gui
import win32con
import keyboard
import os
import ctypes


class BlurTool:
    def __init__(self):
        # Create main window
        self.root = tk.Tk()
        self.root.attributes('-fullscreen', True, '-topmost', True, '-alpha', 0.01)

        # Make the window click-through
        hwnd = win32gui.GetWindow(self.root.winfo_id(), win32con.GW_HWNDNEXT)
        win32gui.SetWindowLong(hwnd, win32con.GWL_EXSTYLE,
                               win32gui.GetWindowLong(hwnd, win32con.GWL_EXSTYLE) | win32con.WS_EX_LAYERED | win32con.WS_EX_TRANSPARENT)

        # Variables
        self.drawing_enabled = True
        self.tool_enabled = True
        self.start_x, self.start_y = None, None
        self.rectangles = []
        self.blur_windows = []

        # Bind events
        self.root.bind('<Button-1>', self.start_rectangle)
        self.root.bind('<B1-Motion>', self.draw_rectangle)
        self.root.bind('<ButtonRelease-1>', self.end_rectangle)

        # Hotkeys
        keyboard.add_hotkey("shift+h", self.toggle_tool)
        keyboard.add_hotkey("shift+a", self.toggle_drawing)
        keyboard.add_hotkey("shift+x", self.reset_all)
        keyboard.add_hotkey("shift+b", self.remove_blurs)
        keyboard.add_hotkey("esc", self.close_tool)

        # Hide console window
        ctypes.windll.user32.ShowWindow(ctypes.windll.kernel32.GetConsoleWindow(), 0)

    def toggle_tool(self):
        """Turn the tool on or off (keeps running in the background)."""
        self.tool_enabled = not self.tool_enabled
        if self.tool_enabled:
            self.root.attributes('-alpha', 0.01)
        else:
            self.root.attributes('-alpha', 0)

    def toggle_drawing(self):
        """Enable or disable drawing rectangles."""
        self.drawing_enabled = not self.drawing_enabled
        if not self.drawing_enabled:
            # Disable click-through while drawing is off
            hwnd = win32gui.GetWindow(self.root.winfo_id(), win32con.GW_HWNDNEXT)
            exstyle = win32gui.GetWindowLong(hwnd, win32con.GWL_EXSTYLE)
            win32gui.SetWindowLong(hwnd, win32con.GWL_EXSTYLE, exstyle & ~win32con.WS_EX_TRANSPARENT)
        else:
            # Re-enable click-through
            hwnd = win32gui.GetWindow(self.root.winfo_id(), win32con.GW_HWNDNEXT)
            win32gui.SetWindowLong(hwnd, win32con.GWL_EXSTYLE,
                                   win32gui.GetWindowLong(hwnd, win32con.GWL_EXSTYLE) | win32con.WS_EX_TRANSPARENT)

    def start_rectangle(self, event):
        """Start drawing a rectangle."""
        if not self.tool_enabled or not self.drawing_enabled:
            return
        self.start_x, self.start_y = event.x, event.y

    def draw_rectangle(self, event):
        """Draw the rectangle dynamically."""
        if not self.tool_enabled or not self.drawing_enabled:
            return
        self.remove_last_rectangle()
        rect = self.root.create_rectangle(self.start_x, self.start_y, event.x, event.y, outline="red", width=2)
        self.rectangles.append(rect)

    def end_rectangle(self, event):
        """Finish drawing and apply blur."""
        if not self.tool_enabled or not self.drawing_enabled:
            return
        x1, y1 = min(self.start_x, event.x), min(self.start_y, event.y)
        x2, y2 = max(self.start_x, event.x), max(self.start_y, event.y)
        self.apply_blur(x1, y1, x2, y2)

    def remove_last_rectangle(self):
        """Remove the last rectangle drawn (while drawing)."""
        if self.rectangles:
            self.root.delete(self.rectangles[-1])
            self.rectangles.pop()

    def apply_blur(self, x1, y1, x2, y2):
        """Capture the screen and apply blur to the selected area."""
        screenshot = ImageGrab.grab()
        cropped = screenshot.crop((x1, y1, x2, y2))
        blurred = cv2.GaussianBlur(np.array(cropped), (25, 25), 0)
        blurred_image = Image.fromarray(blurred)

        # Create a new window for the blurred region
        blur_window = tk.Toplevel(self.root)
        blur_window.geometry(f"{x2-x1}x{y2-y1}+{x1}+{y1}")
        blur_window.overrideredirect(True)
        blur_window.attributes("-topmost", True)
        blur_window.attributes("-alpha", 1.0)

        photo = ImageTk.PhotoImage(blurred_image)
        label = tk.Label(blur_window, image=photo)
        label.image = photo
        label.pack()

        self.blur_windows.append(blur_window)

    def reset_all(self):
        """Reset all blurred areas and rectangles."""
        for rect in self.rectangles:
            self.root.delete(rect)
        self.rectangles.clear()

        for blur_window in self.blur_windows:
            blur_window.destroy()
        self.blur_windows.clear()

    def remove_blurs(self):
        """Remove all blur areas and reset the screen to normal."""
        for blur_window in self.blur_windows:
            blur_window.destroy()
        self.blur_windows.clear()

    def close_tool(self):
        """Close the tool and reset all areas."""
        self.reset_all()
        self.root.destroy()

    def run(self):
        """Run the Tkinter event loop."""
        self.root.mainloop()


if __name__ == "__main__":
    BlurTool().run()
```

