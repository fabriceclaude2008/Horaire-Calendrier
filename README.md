# Agenda scolaire – Automne 2026

Application web personnelle pour gérer ma session au cégep : calendrier des examens et devoirs, horaire hebdomadaire, et estimation de la cote R.

L'application tient dans un seul fichier, `index.html`. Il n'y a rien à installer ni à compiler.

## Fonctionnalités

- **Calendrier mensuel** : numéros de semaine de session (S1 à S16), semaine de relâche, ajout et modification d'un examen ou d'un devoir en un clic.
- **À faire** : liste triée par date, cases à cocher, modification directe des dates, filtres par cours et par période, éléments en retard en rouge.
- **Horaire** : grille du lundi au vendredi, cours modifiables.
- **Notes et cote R** : saisie des notes, de la moyenne et de l'écart type du groupe pour estimer la cote R par cours et globale, avec indicateurs et explications.
- **Documents de cours** : dépôt privé de plans de cours, calendriers et autres fichiers pour les utilisateurs connectés.
- **Compte à rebours** jusqu'au prochain examen.
- **Sauvegarde automatique** dans le navigateur, avec exportation et importation en JSON.
- Thème clair ou sombre.

## Calcul de la cote R

```
Z = (note − moyenne du groupe) ÷ écart type du groupe
R = (Z × IDGZ + IFGZ + 5) × 5
```

- L'IFGZ (force du groupe) et l'IDGZ (dispersion du groupe) sont calculés par le Ministère après la session. Les valeurs de départ (0 et 1) représentent un groupe moyen et sont modifiables.
- La cote R globale est la moyenne des cotes R des cours, pondérée par les unités de chaque cours (par exemple 3-1-3 donne 2,33 unités).
- L'écart type de la note finale est estimé à partir de celui de chaque évaluation et d'une corrélation modifiable entre les évaluations.
- Le résultat est une estimation et ne remplace pas la cote R officielle.

## Utilisation locale

Télécharge `index.html` et ouvre-le dans un navigateur. Aucun serveur n'est nécessaire.

## Déploiement sur Vercel

1. Mets `index.html` (et ce `README.md`) dans un dépôt GitHub.
2. Va sur [vercel.com](https://vercel.com), clique sur **Add New → Project**, puis importe le dépôt.
3. Garde les paramètres par défaut : aucun framework ni commande de build.
4. Clique sur **Deploy**.

Chaque `git push` met le site à jour automatiquement.

## Sauvegarde des données

Les données sont enregistrées dans le `localStorage` du navigateur, donc séparément sur chaque appareil et chaque navigateur. Pour les transférer :

1. Clique sur **Exporter** pour télécharger un fichier JSON.
2. Sur l'autre appareil, clique sur **Importer** et choisis ce fichier.

Vider les données du navigateur efface l'agenda. Exporte régulièrement une copie de sauvegarde.

## Comptes et dépôt privé de documents

L'authentification et les données de compte utilisent le projet Supabase configuré dans `index.html`. Pour activer le dépôt de documents, ouvre le SQL Editor de ce projet Supabase et exécute une fois :

```sql
insert into storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
values (
  'documents-etudiants',
  'documents-etudiants',
  false,
  20971520,
  array[
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'text/calendar',
    'image/png',
    'image/jpeg'
  ]
)
on conflict (id) do update set
  public = false,
  file_size_limit = excluded.file_size_limit,
  allowed_mime_types = excluded.allowed_mime_types;

create policy "Chaque étudiant dépose ses propres documents"
on storage.objects for insert to authenticated
with check (
  bucket_id = 'documents-etudiants'
  and (storage.foldername(name))[1] = auth.uid()::text
);

create policy "Chaque étudiant consulte ses propres documents"
on storage.objects for select to authenticated
using (
  bucket_id = 'documents-etudiants'
  and (storage.foldername(name))[1] = auth.uid()::text
);

create policy "Chaque étudiant supprime ses propres documents"
on storage.objects for delete to authenticated
using (
  bucket_id = 'documents-etudiants'
  and (storage.foldername(name))[1] = auth.uid()::text
);
```

Le dépôt stocke les fichiers de façon privée. Il ne détecte pas encore automatiquement les dates et les devoirs dans un document; ceux-ci doivent être ajoutés dans le calendrier.

## Structure du projet

```
.
├── index.html   # l'application complète (HTML, CSS, JavaScript)
└── README.md
```
