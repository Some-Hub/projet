import matplotlib.pyplot as plt
import matplotlib.patches as patches
import numpy as np
import random

# Configuration des couleurs disponibles
available_colors = [
    '#e91e63',  # rose
    '#9c27b0',  # violet
    '#2196f3',  # bleu
    '#4caf50',  # vert
    '#ff9800',  # orange
    '#f44336',  # rouge
    '#795548',  # marron
    '#607d8b',  # bleu-gris
    '#ffeb3b',  # jaune
    '#9e9e9e',  # gris
    '#3f51b5',  # indigo
    '#00bcd4',  # cyan
    '#8bc34a',  # vert clair
    '#ff5722',  # orange foncé
    '#673ab7',  # violet foncé
    '#009688'  # teal
]


def get_user_input():
    """Collecte les informations sur les nœuds et relations de l'utilisateur"""

    print("=== GÉNÉRATEUR DE DIAGRAMME ERD ===")
    print()

    # Saisie des nœuds
    try:
        num_nodes = int(input("Entrez le nombre de nœuds: "))
        if num_nodes <= 0:
            print("Le nombre de nœuds doit être positif!")
            return None, None
    except ValueError:
        print("Veuillez entrer un nombre valide!")
        return None, None

    nodes = {}
    print(f"\n--- Saisie des {num_nodes} nœuds ---")

    for i in range(num_nodes):
        print(f"\nNœud {i + 1}:")
        name = input(f"  Nom du nœud {i + 1}: ").strip()
        if not name:
            name = f"Node_{i + 1}"

        # Assignation automatique d'une couleur
        color = available_colors[i % len(available_colors)]

        # Position automatique en grille
        cols = int(np.ceil(np.sqrt(num_nodes)))
        x = (i % cols) * 3 + 2
        y = (i // cols) * 3 + 2

        nodes[name] = {
            'pos': (x, y),
            'color': color,
            'text': name
        }

        print(f"  ✓ Nœud '{name}' créé (couleur: {color})")

    # Saisie des relations
    print(f"\n--- Saisie des relations ---")
    try:
        num_relations = int(input("Entrez le nombre de relations: "))
        if num_relations < 0:
            print("Le nombre de relations ne peut pas être négatif!")
            return None, None
    except ValueError:
        print("Veuillez entrer un nombre valide!")
        return None, None

    relations = []
    node_names = list(nodes.keys())

    if num_relations > 0:
        print(f"\nNœuds disponibles: {', '.join(node_names)}")
        print()

        for i in range(num_relations):
            print(f"Relation {i + 1}:")

            # Nœud source
            while True:
                source = input(f"  Nœud source: ").strip()
                if source in node_names:
                    break
                print(f"  ❌ Nœud '{source}' non trouvé. Choisissez parmi: {', '.join(node_names)}")

            # Nœud destination
            while True:
                target = input(f"  Nœud destination: ").strip()
                if target in node_names:
                    break
                print(f"  ❌ Nœud '{target}' non trouvé. Choisissez parmi: {', '.join(node_names)}")

            # Label de la relation
            label = input(f"  Label de la relation (optionnel): ").strip()
            if not label:
                label = f"rel_{i + 1}"

            relations.append((source, target, label))
            print(f"  ✓ Relation '{source}' -> '{target}' ({label}) créée")

    return nodes, relations


def calculate_layout(nodes):
    """Calcule un layout automatique pour éviter les chevauchements"""
    num_nodes = len(nodes)

    if num_nodes <= 1:
        return nodes

    # Disposition en cercle pour un meilleur rendu
    center_x, center_y = 8, 6
    radius = min(6, max(3, num_nodes * 0.5))

    node_names = list(nodes.keys())
    for i, name in enumerate(node_names):
        angle = 2 * np.pi * i / num_nodes
        x = center_x + radius * np.cos(angle)
        y = center_y + radius * np.sin(angle)
        nodes[name]['pos'] = (x, y)

    return nodes


def draw_node(ax, name, node_info):
    """Dessine un nœud sur le graphique"""
    x, y = node_info['pos']
    circle = patches.Circle((x, y), 0.6, facecolor=node_info['color'],
                            edgecolor='black', linewidth=2)
    ax.add_patch(circle)

    # Ajout du texte
    text_color = 'white' if node_info['color'] not in ['#ffeb3b', '#fff3e0'] else 'black'
    ax.text(x, y, node_info['text'], ha='center', va='center',
            fontsize=8, fontweight='bold', color=text_color)


def draw_relation(ax, start_pos, end_pos, label):
    """Dessine une relation entre deux nœuds"""
    x1, y1 = start_pos
    x2, y2 = end_pos

    # Éviter les divisions par zéro
    if abs(x2 - x1) < 0.01 and abs(y2 - y1) < 0.01:
        return

    # Calcul du point de départ et d'arrivée sur le cercle
    dx = x2 - x1
    dy = y2 - y1
    distance = np.sqrt(dx ** 2 + dy ** 2)

    if distance == 0:
        return

    # Normalisation
    dx_norm = dx / distance
    dy_norm = dy / distance

    # Points sur les cercles
    start_x = x1 + 0.6 * dx_norm
    start_y = y1 + 0.6 * dy_norm
    end_x = x2 - 0.6 * dx_norm
    end_y = y2 - 0.6 * dy_norm

    # Dessiner la ligne
    ax.plot([start_x, end_x], [start_y, end_y], 'k-', linewidth=1.5)

    # Ajouter le label au milieu
    mid_x = (start_x + end_x) / 2
    mid_y = (start_y + end_y) / 2
    ax.text(mid_x, mid_y, label, ha='center', va='center',
            fontsize=7, bbox=dict(boxstyle="round,pad=0.3",
                                  facecolor='white', alpha=0.8))


def create_diagram(nodes, relations):
    """Crée et affiche le diagramme ERD"""

    # Calcul du layout optimisé
    nodes = calculate_layout(nodes)

    # Configuration de la figure
    fig, ax = plt.subplots(1, 1, figsize=(16, 12))
    ax.set_xlim(-1, 17)
    ax.set_ylim(-1, 13)
    ax.set_aspect('equal')
    ax.axis('off')

    # Dessiner tous les nœuds
    for name, node_info in nodes.items():
        draw_node(ax, name, node_info)

    # Dessiner toutes les relations
    for start_node, end_node, label in relations:
        if start_node in nodes and end_node in nodes:
            start_pos = nodes[start_node]['pos']
            end_pos = nodes[end_node]['pos']
            draw_relation(ax, start_pos, end_pos, label)

    # Ajouter les titres
    ax.text(8, 12.5, 'Diagramme ERD Généré', fontsize=20, fontweight='bold',
            ha='center')
    ax.text(8, 12, 'Créé automatiquement', fontsize=14,
            ha='center', style='italic')

    # Ajouter le nombre de nœuds et relations
    ax.text(1, 0.5, f'{len(nodes)} nœuds', fontsize=14, fontweight='bold',
            ha='left')
    ax.text(1, 0, f'{len(relations)} relations', fontsize=14, fontweight='bold',
            ha='left')

    # Dessiner le cadre principal
    rect = patches.Rectangle((0, 0), 16, 12, linewidth=3,
                             edgecolor='black', facecolor='none')
    ax.add_patch(rect)

    # Affichage du graphique
    plt.tight_layout()
    plt.show()


def main():
    """Fonction principale"""
    try:
        # Collecte des données utilisateur
        nodes, relations = get_user_input()

        if nodes is None or relations is None:
            print("❌ Erreur dans la saisie. Arrêt du programme.")
            return

        print("\n=== GÉNÉRATION DU DIAGRAMME ===")
        print("📊 Création du diagramme en cours...")

        # Création et affichage du diagramme
        create_diagram(nodes, relations)

        print("✅ Diagramme généré avec succès!")
        print(f"📈 Résumé: {len(nodes)} nœuds, {len(relations)} relations")

        # Option de sauvegarde
        save = input("\nVoulez-vous sauvegarder le diagramme? (o/n): ").lower().strip()
        if save in ['o', 'oui', 'y', 'yes']:
            filename = input("Nom du fichier (sans extension): ").strip()
            if not filename:
                filename = "diagramme_erd"

            plt.savefig(f'{filename}.png', dpi=300, bbox_inches='tight')
            print(f"💾 Diagramme sauvegardé sous '{filename}.png'")

    except KeyboardInterrupt:
        print("\n\n❌ Arrêt du programme par l'utilisateur.")
    except Exception as e:
        print(f"❌ Erreur inattendue: {e}")


if _name_ == "_main_":
    main()
import networkx as nx
import matplotlib.pyplot as plt

# Création du graphe
G = nx.DiGraph()

# Ajout des entités principales
entities = [
    "University1", "University2",
    "Faculté des Lettres Modernes", "Faculté d'Informatique", "Faculté SEA",
    "Teacher1", "Teacher2",
    "Student1", "Student2",
    "Session Course1", "Session Course2",
    "City1", "City2",
    "Cameroun", "Burkina Faso",
    "Afrique"
