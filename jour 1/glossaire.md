# Glossaire — Architecture & Analyse

## C4 Model
Cadre de modélisation de l'architecture logicielle organisé en 4 niveaux d'abstraction : **Context, Container, Component et Code**. Créé par Simon Brown, il permet de visualiser et communiquer l'architecture d'un système de manière progressive, du plus général au plus détaillé.

---

## C1 — Context
Premier niveau du C4 Model. Le diagramme de contexte représente le système dans son environnement : les utilisateurs qui l'utilisent et les systèmes externes avec lesquels il interagit. C'est la vue la plus abstraite et la plus accessible à tous les interlocuteurs, techniques ou non.

---

## Système
Ensemble cohérent de composants logiciels (applications, bases de données, services) qui fournit de la valeur à ses utilisateurs. Dans le C4 Model, le système est l'entité centrale du diagramme C1 — c'est ce que l'on cherche à décrire et à construire.

---

## Acteur
Entité externe — humaine ou système tiers — qui interagit avec le système. Dans le C4 Model (C1), les acteurs sont représentés en dehors du périmètre du système. En UML, un acteur peut initier des cas d'utilisation ou en être le bénéficiaire.

> **Types :** acteur humain (client, administrateur) ou système externe (API bancaire, service SMS).

---

## User Story
Description courte d'une fonctionnalité du point de vue de l'utilisateur, selon le format :

> *"En tant que [rôle], je veux [action] afin de [bénéfice]."*

Outil central des méthodes agiles pour exprimer les besoins de façon centrée sur l'utilisateur et sa valeur métier.

---

## Use Case
Cas d'utilisation : description structurée d'une interaction entre un acteur et le système pour atteindre un objectif. Plus formel qu'une User Story, un Use Case détaille les scénarios principal et alternatifs, les préconditions et postconditions. Issu de la méthode UML.

> **Différence clé :** la User Story dit *quoi* et *pourquoi* ; le Use Case décrit *comment* le système répond étape par étape.

## diagramme de contexte C1

```mermaid
graph TD
    U["👤 Utilisateur"]
    A["👤 Admin"]
    C["👤 Collaborateur"]

    S["💻 MyToDo<br/>Application de gestion de tâches"]

    E["⚙️ Service Email<br/>[Système externe]"]
    N["⚙️ Notification Service<br/>[Système externe]"]

    U -- "Créé un compte" --> S
    U -- "Gère ses tâches" --> S
    A -- "Gère les groupes et les attributions" --> S
    C -- "Soumet des tâches" --> S

    S -- "Envoie des emails" --> E
    S -- "Envoie des notifications" --> N