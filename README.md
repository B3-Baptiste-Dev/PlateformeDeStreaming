# Conception de la Base de Données pour une Plateforme de Streaming

## Dictionnaire de Données

| Table        | Donnée        | Type          | Taille       | Obligatoire |
|--------------|---------------|---------------|--------------|-------------|
| Personne     | idPersonne    | INT           |              | Oui         |
|              | nom           | VARCHAR       | 100          | Oui         |
|              | prénom        | VARCHAR       | 100          | Oui         |
|              | dateNaissance | DATE          |              | Oui         |
| Réalisateur  | idRéalisateur | INT           |              | Oui         |
|              | idPersonne    | INT           |              | Oui         |
| Acteur       | idActeur      | INT           |              | Oui         |
|              | idPersonne    | INT           |              | Oui         |
| Film         | idFilm        | INT           |              | Oui         |
|              | titre         | VARCHAR       | 255          | Oui         |
|              | durée         | INT           |              | Oui         |
|              | annéeSortie   | YEAR          |              | Oui         |
|              | idRéalisateur | INT           |              | Oui         |
| FilmActeur   | idFilm        | INT           |              | Oui         |
|              | idActeur      | INT           |              | Oui         |
|              | rôle          | VARCHAR       | 100          | Oui         |
| Utilisateur  | idUtilisateur | INT           |              | Oui         |
|              | nom           | VARCHAR       | 100          | Oui         |
|              | prénom        | VARCHAR       | 100          | Oui         |
|              | email         | VARCHAR       | 100          | Oui         |
|              | motDePasse    | VARCHAR       | 255          | Oui         |
|              | rôle          | VARCHAR       | 100          | Oui         |
| FilmPréféré  | idUtilisateur | INT           |              | Oui         |
|              | idFilm        | INT           |              | Oui         |
| Genre        | idGenre       | INT           |              | Oui         |
|              | nom           | VARCHAR       | 100          | Oui         |
| FilmGenre    | idFilm        | INT           |              | Oui         |
|              | idGenre       | INT           |              | Oui         |





## Requêtes SQL pour la Création des Tables

```sql
CREATE DATABASE IF NOT EXISTS PlateformeDB;
USE PlateformeDB;

CREATE TABLE Personne (
    idPersonne INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100) NOT NULL,
    prénom VARCHAR(100) NOT NULL,
    dateNaissance DATE NOT NULL
) ENGINE=InnoDB;

CREATE TABLE Réalisateur (
    idRéalisateur INT AUTO_INCREMENT PRIMARY KEY,
    idPersonne INT,
    FOREIGN KEY (idPersonne) REFERENCES Personne(idPersonne) ON DELETE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Acteur (
    idActeur INT AUTO_INCREMENT PRIMARY KEY,
    idPersonne INT,
    FOREIGN KEY (idPersonne) REFERENCES Personne(idPersonne) ON DELETE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Film (
    idFilm INT AUTO_INCREMENT PRIMARY KEY,
    titre VARCHAR(255) NOT NULL,
    durée INT,
    annéeSortie YEAR,
    idRéalisateur INT,
    FOREIGN KEY (idRéalisateur) REFERENCES Réalisateur(idRéalisateur) ON DELETE SET NULL
) ENGINE=InnoDB;

CREATE TABLE FilmActeur (
    idFilm INT,
    idActeur INT,
    rôle VARCHAR(100),
    PRIMARY KEY (idFilm, idActeur),
    FOREIGN KEY (idFilm) REFERENCES Film(idFilm) ON DELETE CASCADE,
    FOREIGN KEY (idActeur) REFERENCES Acteur(idActeur) ON DELETE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Utilisateur (
    idUtilisateur INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100) NOT NULL,
    prénom VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    motDePasse VARCHAR(255) NOT NULL,
    rôle VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

CREATE TABLE FilmPréféré (
    idUtilisateur INT,
    idFilm INT,
    PRIMARY KEY (idUtilisateur, idFilm),
    FOREIGN KEY (idUtilisateur) REFERENCES Utilisateur(idUtilisateur) ON DELETE CASCADE,
    FOREIGN KEY (idFilm) REFERENCES Film(idFilm) ON DELETE CASCADE
) ENGINE=InnoDB;

CREATE TABLE Genre (
    idGenre INT AUTO_INCREMENT PRIMARY KEY,
    nom VARCHAR(100) UNIQUE NOT NULL
) ENGINE=InnoDB;

CREATE TABLE FilmGenre (
    idFilm INT,
    idGenre INT,
    PRIMARY KEY (idFilm, idGenre),
    FOREIGN KEY (idFilm) REFERENCES Film(idFilm) ON DELETE CASCADE,
    FOREIGN KEY (idGenre) REFERENCES Genre(idGenre) ON DELETE CASCADE
) ENGINE=InnoDB;

```

## Jeu de données

```sql
INSERT INTO Personne (nom, prénom, dateNaissance) VALUES
('Downey', 'Robert', '1965-04-04'),
('Johansson', 'Scarlett', '1984-11-22'),
('Hemsworth', 'Chris', '1983-08-11');

INSERT INTO Réalisateur (idPersonne) VALUES (1);

INSERT INTO Acteur (idPersonne) VALUES (1), (2), (3);

INSERT INTO Film (titre, durée, annéeSortie, idRéalisateur) VALUES
('Avengers: Infinity War', 149, 2018, 1),
('Avengers: Endgame', 181, 2019, 1);

INSERT INTO FilmActeur (idFilm, idActeur, rôle) VALUES
(1, 1, 'Tony Stark / Iron Man'),
(1, 2, 'Natasha Romanoff / Black Widow'),
(2, 3, 'Thor');

INSERT INTO Utilisateur (nom, prénom, email, motDePasse, rôle) VALUES
('User', 'Example', 'user@example.com', 'securepassword', 'viewer');

INSERT INTO FilmPréféré (idUtilisateur, idFilm) VALUES (1, 1), (1, 2);

INSERT INTO Genre (nom) VALUES ('Action'), ('Science-Fiction');

INSERT INTO FilmGenre (idFilm, idGenre) VALUES
(1, 1), (1, 2),
(2, 1), (2, 2);
```

## Exercices

### 1.Les titres et dates de sortie des films du plus récent au plus ancien
```sql
SELECT titre, annéeSortie FROM Film ORDER BY annéeSortie DESC;
```

### 2. Les noms, prénoms et âges des acteurs/actrices de plus de 30 ans dans l'ordre alphabétique
```sql
SELECT p.nom, p.prénom, TIMESTAMPDIFF(YEAR, p.dateNaissance, CURDATE()) AS Age
FROM Personne p
JOIN Acteur a ON p.idPersonne = a.idPersonne
WHERE TIMESTAMPDIFF(YEAR, p.dateNaissance, CURDATE()) > 30
ORDER BY p.nom, p.prénom;
```

### 3. La liste des acteurs/actrices principaux pour un film donné
```sql
SELECT p.nom, p.prénom, fa.rôle
FROM FilmActeur fa
JOIN Acteur a ON fa.idActeur = a.idActeur
JOIN Personne p ON a.idPersonne = p.idPersonne
WHERE fa.idFilm =1; 
```

### 4. La liste des films pour un acteur/actrice donné
```sql
SELECT f.titre, f.annéeSortie
FROM FilmActeur fa
JOIN Film f ON fa.idFilm = f.idFilm
WHERE fa.idActeur = 1;
```

### 5. Ajouter un film
```sql
INSERT INTO Film (titre, durée, annéeSortie, idRéalisateur) VALUES ('Nom du film', Durée en minutes, Année de sortie, ID du réalisateur);
```

### 6. Ajouter un acteur/actrice
```sql
INSERT INTO Personne (nom, prénom, dateNaissance) VALUES ('Nom', 'Prénom', 'Date de naissance');
INSERT INTO Acteur (idPersonne) VALUES (LAST_INSERT_ID());
```

### 7. Modifier un film
```sql
UPDATE Film SET titre = 'Nouveau titre', durée = Nouvelle durée, annéeSortie = Nouvelle année, idRéalisateur = Nouvel ID réalisateur WHERE idFilm = 1;
```

### 8. Supprimer un acteur/actrice
```sql
DELETE FROM Acteur WHERE idActeur = 1;
DELETE FROM Personne WHERE idPersonne = 1;
```

### 9. Afficher les 3 derniers acteurs/actrices ajouté(e)s
```sql
SELECT p.nom, p.prénom
FROM Acteur a
JOIN Personne p ON a.idPersonne = p.idPersonne
ORDER BY a.idActeur DESC
LIMIT 3;
```

### 10. Afficher le film le plus ancien
```sql
SELECT titre, annéeSortie FROM Film ORDER BY annéeSortie ASC LIMIT 1;
```

### 11. Afficher l’acteur le plus jeune
```sql
SELECT p.nom, p.prénom, p.dateNaissance
FROM Acteur a
JOIN Personne p ON a.idPersonne = p.idPersonne
ORDER BY p.dateNaissance DESC
LIMIT 1;
```

### 12. Compter le nombre de films réalisés en 1990
```sql
SELECT COUNT(*) FROM Film WHERE annéeSortie = 1990;
```

### 13. Faire la somme de tous les acteurs ayant joué dans un film
```sql
SELECT COUNT(DISTINCT idActeur) FROM FilmActeur;
```

![MCD](https://github.com/B3-Baptiste-Dev/PlateformeDeStreaming/blob/main/mcd.svg)

![MLD](https://github.com/B3-Baptiste-Dev/PlateformeDeStreaming/blob/main/mld.svg)
