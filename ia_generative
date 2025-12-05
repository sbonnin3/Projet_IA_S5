# IA Générative Clash Royale : Trouver le meilleur deck pour battre un deck donné
# Interface similaire, fonctionnement :
# - L'utilisateur choisit 8 cartes (son deck)
# - L'IA parcourt tous les combats historiques et cherche les decks adverses les plus efficaces
# - Elle reconstruit le deck le plus joué avec les meilleures stats contre ce deck

import tkinter as tk
from tkinter import ttk, messagebox
import csv
from collections import Counter, defaultdict

CARTES_FILE = "dataset/cartes.csv"
DONNEES_COMBATS = "dataset/combats_joueurs.csv"

# ================================================================
# Charger cartes.csv
# ================================================================
def load_cards():
    cards = []
    with open(CARTES_FILE, "r", encoding="utf-8") as f:
        reader = csv.reader(f)
        next(reader)
        for row in reader:
            cid = int(row[0])
            name = row[1]
            cards.append((cid, name))
    return cards

# ================================================================
# IA générative : trouver le meilleur deck pour battre un deck donné
# ---------------------------------------------------------------
# Principe :
# 1. L'utilisateur fournit un deck (8 cartes)
# 2. On parcourt tous les combats historiques
# 3. Chaque fois qu'un deck A bat un deck B et que B ressemble fortement au deck utilisateur,
#    on ajoute les cartes de A à un compteur
# 4. À la fin, on prend les 8 cartes les plus fréquentes comme "deck optimal adversaire"
# ================================================================

def generer_deck_anti(deck_user, precision):
    compteur_cartes = Counter()
    combats_matches = 0

    with open(DONNEES_COMBATS, "r", encoding="utf-8") as f:
        reader = csv.reader(f)
        next(reader)

        for row in reader:
            gagnant = row[2:10]
            perdant = row[12:20]

            # Cas où le perdant ressemble à l'utilisateur → prendre gagnant
            if len(set(deck_user) & set(perdant)) >= precision:
                compteur_cartes.update(gagnant)
                combats_matches += 1

            # Cas inverse : gagnant ressemble à l'utilisateur → prendre perdant
            if len(set(deck_user) & set(gagnant)) >= precision:
                compteur_cartes.update(perdant)
                combats_matches += 1

    if combats_matches == 0:
        return [], 0

    # Top 8 cartes les plus efficaces contre deck_user
    best_8 = [c for c, _ in compteur_cartes.most_common(8)]
    return best_8, combats_matches

# ================================================================
# Interface graphique moderne
# ================================================================
class IA_Generative_App:

    def __init__(self, root):
        self.root = root
        self.cards = load_cards()
        self.deck_vars = []

        root.title("IA Générative Clash Royale - Deck Contre Deck")
        root.geometry("900x700")
        root.configure(bg="#1e1e1e")
        self.build_ui()

    # ----------------------------------------------------------
    def build_ui(self):
        title = tk.Label(self.root, text="IA Générative - Trouver le deck parfait pour te battre",
                         fg="white", bg="#1e1e1e", font=("Arial", 20, "bold"))
        title.pack(pady=10)

        frame = tk.Frame(self.root, bg="#1e1e1e")
        frame.pack(fill="both", expand=True)

        canvas = tk.Canvas(frame, bg="#1e1e1e", highlightthickness=0)
        canvas.pack(side="left", fill="both", expand=True)

        scrollbar = ttk.Scrollbar(frame, orient="vertical", command=canvas.yview)
        scrollbar.pack(side="right", fill="y")

        canvas.configure(yscrollcommand=scrollbar.set)

        self.table_frame = tk.Frame(canvas, bg="#1e1e1e")
        canvas_window = canvas.create_window((0, 0), window=self.table_frame, anchor="nw")

        self.table_frame.bind(
            "<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )
        canvas.bind(
            "<Configure>", lambda e: canvas.itemconfig(canvas_window, width=e.width)
        )

        headers = ["ID", "Carte", "Ton Deck"]
        for c, h in enumerate(headers):
            tk.Label(self.table_frame, text=h, fg="white", bg="#1e1e1e",
                     font=("Arial", 12, "bold")).grid(row=0, column=c, padx=15, pady=5)

        for i, (cid, name) in enumerate(self.cards):
            row = i + 1

            tk.Label(self.table_frame, text=str(cid), fg="white", bg="#1e1e1e").grid(row=row, column=0)
            tk.Label(self.table_frame, text=name, fg="white", bg="#1e1e1e", width=40, anchor="w").grid(row=row, column=1)

            var = tk.BooleanVar(value=False)
            cb = tk.Checkbutton(self.table_frame, variable=var, bg="#1e1e1e", activebackground="#333",
                                command=self.check_selection)
            cb.grid(row=row, column=2)
            self.deck_vars.append(var)

        # Precision
        precision_frame = tk.Frame(self.root, bg="#1e1e1e")
        precision_frame.pack(pady=10)

        tk.Label(precision_frame, text="Précision (1 à 8) :", fg="white", bg="#1e1e1e",
                 font=("Arial", 13)).pack(side="left")

        self.precision_var = tk.IntVar(value=5)
        box = ttk.Combobox(precision_frame, textvariable=self.precision_var, width=5,
                           values=list(range(1, 9)))
        box.pack(side="left", padx=5)

        # Résultats
        self.result_frame = tk.LabelFrame(self.root, text="Deck généré par l'IA", fg="white", bg="#1e1e1e",
                                          font=("Arial", 12, "bold"), padx=10, pady=10)
        self.result_frame.pack(fill="x", padx=10, pady=10)

        self.result_label = tk.Label(self.result_frame, text="Sélectionne ton deck (8 cartes).",
                                     fg="white", bg="#1e1e1e", font=("Arial", 14))
        self.result_label.pack()

    # ----------------------------------------------------------
    def check_selection(self):
        if sum(v.get() for v in self.deck_vars) > 8:
            messagebox.showwarning("Limite", "Ton deck ne peut contenir que 8 cartes.")
            self.reset_excess()
        if sum(v.get() for v in self.deck_vars) == 8:
            self.run_ia()

    def reset_excess(self):
        count = 0
        for v in self.deck_vars:
            if v.get():
                count += 1
                if count > 8:
                    v.set(False)

    # ----------------------------------------------------------
    def run_ia(self):
        precision = self.precision_var.get()
        deck_user = [self.cards[i][1] for i, v in enumerate(self.deck_vars) if v.get()]

        suggested, combats = generer_deck_anti(deck_user, precision)

        if combats == 0:
            self.result_label.config(text="Aucun combat trouvé correspondant à ton deck.")
            return

        txt = "Deck optimal pour te battre :\n" + "\n".join(suggested)
        txt += f"\n\nCombats analysés : {combats}"

        self.result_label.config(text=txt)

# ================================================================
# Lancement
# ================================================================
if __name__ == "__main__":
    root = tk.Tk()
    app = IA_Generative_App(root)
    root.mainloop()