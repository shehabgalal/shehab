import tkinter as tk
from tkinter import ttk, messagebox
from PIL import Image, ImageTk
import sys
import os

class SWOTAnalyzerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("SWOT Analyzer Pro+")
        self.root.geometry("1000x800")
        
        self.style = ttk.Style()
        self.style.configure('TNotebook.Tab', font=('Arial', 10, 'bold'))
        self.style.configure('Accent.TButton', 
                           font=('Arial', 12, 'bold'), 
                           foreground='white',
                           background='#0078D4')
        
        self.create_widgets()
        
    def create_category_tab(self, notebook, category, questions):
        frame = ttk.Frame(notebook)
        notebook.add(frame, text=category)
        
        for i, question in enumerate(questions):
            row = ttk.Frame(frame)
            row.pack(fill='x', padx=10, pady=5)
            
            ttk.Label(row, text=question, width=70, wraplength=600).pack(side='left')
            
            self.sliders[f"{category}_{i}"] = tk.IntVar(value=3)
            # 
            scale = ttk.Scale(
                row, 
                from_=1, 
                to=5, 
                variable=self.sliders[f"{category}_{i}"],
                command=lambda v, var=self.sliders[f"{category}_{i}"]: var.set(round(float(v)))
            )
            scale.pack(side='right', fill='x', expand=True)
            
            value_label = ttk.Label(row, text="3", width=3)
            value_label.pack(side='right', padx=5)
            # إصلاح الأقواس هنا
            scale.bind("<Motion>", lambda e, lbl=value_label: lbl.config(text=str(round(float(e.widget.get())))))
        
    def create_widgets(self):
        self.sliders = {}
        
        notebook = ttk.Notebook(self.root)
        notebook.pack(fill='both', expand=True, padx=10, pady=10)
        
        categories = {
            'Strengths (1=Weak, 5=Excellent)': [
                "Strong brand reputation", "Skilled workforce", "Unique technology/patents",
                "Customer loyalty", "Effective distribution channels", "Strong financial reserves",
                "Efficient supply chain", "Innovative culture", "High-quality products",
                "Strong online presence"
            ],
            'Weaknesses (1=Minor, 5=Critical)': [
                "High operational costs", "Weak digital marketing", "Dependence on key customers",
                "Outdated technology infrastructure", "Poor risk management", "Limited R&D budget",
                "High employee turnover", "Slow decision-making processes", "Limited geographic reach",
                "Inconsistent product quality"
            ],
            'Opportunities (1=Weak, 5=Excellent)': [
                "New market entry", "Growing product demand", "Strategic partnerships",
                "Available funding", "New technology adoption", "Emerging markets expansion",
                "Technological advancements", "Changing consumer trends", "Government incentives",
                "Acquisition opportunities"
            ],
            'Threats (1=Minor, 5=Critical)': [
                "Price competition", "Regulatory changes", "Raw material price fluctuations",
                "Cybersecurity threats", "Supplier dependence", "Economic downturn",
                "Changing consumer preferences", "Environmental regulations", "Potential lawsuits",
                "Natural disasters risk"
            ]
        }
        
        for cat, questions in categories.items():
            self.create_category_tab(notebook, cat, questions)
            
        btn_frame = ttk.Frame(self.root)
        btn_frame.pack(pady=20)
        ttk.Button(
            btn_frame, 
            text="Run Strategic Analysis", 
            command=self.analyze,
            style='Accent.TButton'
        ).pack(padx=20, ipadx=10, ipady=5)
    
    def analyze(self):
        try:
            scores = {
                'S': sum(self.sliders[f"Strengths (1=Weak, 5=Excellent)_{i}"].get() for i in range(10)),
                'W': sum(self.sliders[f"Weaknesses (1=Minor, 5=Critical)_{i}"].get() for i in range(10)),
                'O': sum(self.sliders[f"Opportunities (1=Weak, 5=Excellent)_{i}"].get() for i in range(10)),
                'T': sum(self.sliders[f"Threats (1=Minor, 5=Critical)_{i}"].get() for i in range(10))
            }
            
            strategy_info = self.determine_strategy(scores)
            self.show_results(scores, strategy_info)
            
        except Exception as e:
            messagebox.showerror("Error", f"An error occurred: {str(e)}")
    
    def determine_strategy(self, scores):
        S = scores['S']
        W = scores['W']
        O = scores['O']
        T = scores['T']
        
        sw_relation = S > W
        ot_relation = O > T

        if sw_relation and ot_relation:
            quadrant = "SO (Growth Zone)"
            strategy = "Aggressive Growth: Leverage strengths to maximize opportunities"
        elif sw_relation and not ot_relation:
            quadrant = "ST (Maintenance Zone)"
            strategy = "Defensive Strategy: Use strengths to mitigate threats"
        elif not sw_relation and ot_relation:
            quadrant = "WO (Harvest Zone)"
            strategy = "Optimization: Focus on opportunities despite weaknesses"
        else:
            quadrant = "WT (Divest Zone)"
            strategy = "Survival Strategy: Minimize weaknesses and threats"
        
        return {
            'quadrant': quadrant,
            'strategy': strategy,
            'sw_ratio': "S > W" if sw_relation else "W ≥ S",
            'ot_ratio': "O > T" if ot_relation else "T ≥ O"
        }
    
    def show_results(self, scores, strategy_info):
        result_window = tk.Toplevel()
        result_window.title("Analysis Results")
        result_window.geometry("1200x900")
        
        main_frame = ttk.Frame(result_window)
        main_frame.pack(fill='both', expand=True, padx=20, pady=20)
        
        ttk.Label(main_frame, 
                 text="Strategic Analysis Results",
                 font=('Arial', 18, 'bold'),
                 foreground='#1E3F66').pack(pady=10)
        
        comparison_frame = ttk.Frame(main_frame)
        comparison_frame.pack(pady=15, fill='x')
        
        ttk.Label(comparison_frame, 
                 text=f"Strengths vs Weaknesses: {strategy_info['sw_ratio']}",
                 font=('Arial', 12, 'bold'),
                 foreground='green' if strategy_info['sw_ratio'] == 'S > W' else 'red'
                ).pack(side='left', padx=20)
        
        ttk.Label(comparison_frame, 
                 text=f"Opportunities vs Threats: {strategy_info['ot_ratio']}",
                 font=('Arial', 12, 'bold'),
                 foreground='green' if strategy_info['ot_ratio'] == 'O > T' else 'red'
                ).pack(side='right', padx=20)
        
        ttk.Label(main_frame, 
                 text=f"Current Quadrant: {strategy_info['quadrant']}",
                 font=('Arial', 14, 'bold'),
                 foreground='#1E3F66').pack(pady=15)
        
        self.show_strategy_image(main_frame, strategy_info['quadrant'])
        
        metrics_frame = ttk.Frame(main_frame)
        metrics_frame.pack(pady=20)
        
        metrics = [
            ("Strengths", scores['S'], 'green'),
            ("Weaknesses", scores['W'], 'red'),
            ("Opportunities", scores['O'], 'green'),
            ("Threats", scores['T'], 'red')
        ]
        
        for metric in metrics:
            row = ttk.Frame(metrics_frame)
            row.pack(fill='x', pady=5)
            ttk.Label(row, text=metric[0], width=15, anchor='w').pack(side='left')
            ttk.Label(row, 
                     text=f"{metric[1]}/50", 
                     foreground=metric[2], 
                     font=('Arial', 12, 'bold')).pack(side='right')
        
        ttk.Label(main_frame, 
                 text="Recommended Strategy:",
                 font=('Arial', 14, 'bold')).pack(pady=(25,5))
        
        ttk.Label(main_frame, 
                 text=strategy_info['strategy'],
                 wraplength=900,
                 font=('Arial', 12),
                 foreground='#2A5C8A').pack(pady=10)
        
        ttk.Button(main_frame, 
                  text="Close", 
                  command=result_window.destroy,
                  style='Accent.TButton').pack(pady=20)
    
    def show_strategy_image(self, parent, quadrant):
        image_map = {
            "SO (Growth Zone)": "so_zone.png",
            "ST (Maintenance Zone)": "st_zone.png",
            "WO (Harvest Zone)": "wo_zone.png",
            "WT (Divest Zone)": "wt_zone.png"
        }
        
        try:
            image_path = resource_path(image_map[quadrant])
            img = Image.open(image_path)
            img = img.resize((600, 400), Image.Resampling.LANCZOS)
            photo = ImageTk.PhotoImage(img)
            
            img_label = ttk.Label(parent, image=photo)
            img_label.image = photo
            img_label.pack(pady=20)
            
        except Exception as e:
            messagebox.showerror("Image Error", f"Failed to load image: {str(e)}")

def resource_path(relative_path):
    try:
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)

if __name__ == "__main__":
    root = tk.Tk()
    try:
        root.iconbitmap(resource_path('swot_icon.ico'))
    except Exception as e:
        print(f"Icon error: {e}")
    app = SWOTAnalyzerApp(root)
    root.mainloop()
name: Build Executable
     on: [push]
     jobs:
       build:
         runs-on: windows-latest
         steps:
           - uses: actions/checkout@v2
           - name: Install dependencies
             run: pip install pyinstaller
           - name: Build EXE
             run: pyinstaller --onefile --windowed --icon=swot_icon.ico SWOT_Analyzer.py
           - name: Upload Artifact
             uses: actions/upload-artifact@v2
             with:
               name: SWOT_Analyzer.exe
               path: dist/SWOT_Analyzer.exe