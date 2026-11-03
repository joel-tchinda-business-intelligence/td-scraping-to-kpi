# Réflexions TD3 — Scraping to KPI

## 1. Pourquoi un échec silencieux de robots.txt est pire qu'un échec visible
Si le script plante, on voit tout de suite qu'il y a un problème et on le corrige. Si la vérification renvoie « interdit » sans aucune erreur, on ne sait pas si le site refuse vraiment ou si c'est la vérification qui n'a pas marché. Pour une étape dont le seul rôle est de prouver qu'on a le droit de récupérer les données, ne pas savoir si elle a vraiment fonctionné revient à ne pas pouvoir justifier la provenance des données. Un échec visible, lui, laisse une trace qu'on peut expliquer.

## 2. Données France Travail vs échantillon jobspy : différence de risque
Les données France Travail sont publiques avec une licence qui autorise leur réutilisation, donc on peut les publier sans problème. Les offres récupérées par jobspy viennent d'Indeed, dont les conditions d'utilisation interdisent de redistribuer le contenu; les mettre dans un dépôt public, c'est les redistribuer à tout le monde. Avec un dépôt privé, le risque est plus faible car personne d'autre n'y a accès, mais ça reste une copie qu'on n'a pas le droit de partager. C'est pour ça que seul le KPI agrégé est publié, et que le CSV reste sur mon ordinateur.

## 3. Pourquoi extraire des compétences est une bonne tâche pour un LLM, et pas le suivi des candidatures
Les offres sont du texte libre, écrit différemment par chaque entreprise, en français ou en anglais. un LLM est bon pour comprendre ce genre de texte et en sortir une liste, et une petite erreur est acceptable puisqu'on peut la vérifier en relisant l'offre. Par contre, savoir à quelles offres j'ai vraiment postulé est un fait que je connais déjà exactement; demander à un LLM de le deviner ajouterait des erreurs possibles sans rien apporter. Un simple tableau suffit.

## 4. Incohérence de format prévue, et ce que j'ai trouvé dans extracted.json
Je m'attendais à ce que la même compétence soit écrite avec des majuscules différentes ou sous des noms un peu différents, puisque chaque offre est traitée par un appel indépendant. C'est bien ce que j'ai trouvé : "Generative AI" dans les offres 1 et 2, et "generative AI" dans l'offre 5. J'ai aussi "CI/CD", "GitLab CI/CD" et "CI/CD pipelines", ou "AI" et "AI/ML". Si on compte sans nettoyer, une même compétence est coupée en plusieurs lignes et paraît moins demandée qu'elle ne l'est.
On le voit dans kpi.json, "Generative AI" n'est compté que 2 fois au lieu de 3. J'ai aussi remarqué que les offres 1 et 2 sont la même annonce publiée deux fois, ce qui fait apparaître plusieurs compétences (Computer vision, 360° video, SaaS...) dans le top 10 uniquement à cause de ce doublon.

## 5. Pourquoi la relecture croisée est du human-in-the-loop
C'est un humain qui compare la sortie du modèle au texte d'origine, au lieu de faire confiance au modèle. En relisant, j'ai trouvé une vraie erreur : l'offre BCG X n'a aucune compétence extraite, alors qu'elle demande Python et AWS. La cause est que le script ne lit que les 1 500 premiers caractères, qui ne parlent que de l'entreprise. Au total, Python apparaît dans 8 offres sur 12 mais n'a été détecté que dans une seule. Cette vérification doit se faire sur extracted.json, car on y voit chaque offre séparément ; dans kpi.json, il ne reste qu'un pourcentage et des totaux, et l'erreur devient invisible.

## 6. Ma compétence cible, et quel KPI je trouve le plus fiable
Pour le comité pédagogique de l'ENSIM, je suivrais d'abord CI/CD, parce que c'est une compétence de plus en plus demandée en développement logiciel et en DevOps, et qu'elle est encore peu enseignée en pratique à l'école.

Résultats : Python est cité dans 8 % des offres (1 sur 12) et CI/CD dans 25 % (3 sur 12).

Je ne ferais confiance à aucun des deux pour prendre une vraie décision. En relisant les offres, j'ai vu que Python est en fait demandé dans 8 offres sur 12, mais le script ne lit que les 1 500 premiers caractères, souvent consacrés à la présentation de l'entreprise, donc le KPI Python est très sous-estimé. En plus, 12 offres prises au hasard sur Indeed, c'est trop peu et pas forcément représentatif des postes visés par les étudiants de l'ENSIM. Le KPI CI/CD paraît meilleur seulement parce que cette compétence apparaissait plus tôt dans le texte, pas parce qu'elle est vraiment plus demandée que Python.

## 7. Ce qu'on peut ou non reconstruire à partir de kpi.json
À partir de kpi.json seul, on ne peut pas retrouver les offres ni les entreprises. il ne contient qu'un pourcentage, des totaux et une liste de compétences avec leur nombre. C'est un choix voulu, le titre et l'entreprise existent dans extracted.json, mais la fonction compute_kpi() ne les recopie jamais dans le résultat, elle ne garde que des comptages.

Par contre, ce n'est pas un anonymat parfait. Avec seulement 12 offres, certaines compétences très précises du top 10, comme "360° video" ou "HIL test benches", correspondent à une seule annonce et peuvent donner un indice sur le type d'entreprise qui l'a publiée. Avec plus d'offres, ou en ne publiant que les compétences citées au moins 2 ou 3 fois, ce risque serait plus faible.

## 8. À qui s'adresse vraiment ce dashboard
En l'état, la page s'adresse surtout à moi ou au professeur : le titre « Scraping to KPI lab » et la phrase qui renvoie vers extracted.json et compute_kpi.py parlent de la méthode, pas d'une décision. Un membre du comité pédagogique ne saurait pas quoi en conclure, et il ne pourrait même pas ouvrir extracted.json puisque ce fichier n'est pas publié. Je changerais la description en : « Part des offres d'emploi d'ingénieur qui demandent des compétences CI/CD, pour aider l'ENSIM à décider quoi renforcer dans ses cours - échantillon de 12 offres »

## 9. Mon deuxième KPI, et lequel je trouve le plus fiable
Mon deuxième KPI mesure la part des offres qui citent au moins un fournisseur cloud (AWS, Azure ou Google Cloud) : 17 % (2 offres sur 12), contre 25 % pour CI/CD.

Je ferais un peu plus confiance au KPI cloud pour une décision, parce qu'il regroupe plusieurs mots-clés : il dépend moins de la façon exacte dont le modèle écrit une compétence, et il répond à une question plus utile pour l'école (« faut-il enseigner le cloud ? ») plutôt qu'à un seul outil. Mais les deux restent fragiles; seulement 12 offres, et le script ne lit que les 1 500 premiers caractères, donc il rate des mentions. il faudrait analyser le texte complet de beaucoup plus d'offres.