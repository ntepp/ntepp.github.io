---
layout: post
title: "Comment j'ai préparé et réussi l'examen Claude Certified Architect – Foundations d'Anthropic"
subtitle: "Ce que l'examen teste vraiment, les ressources qui m'ont le plus aidé, et ce que je changerais la prochaine fois."
tags: [IA, Claude Code, Certification]
comments: false
social-share: true
---

*Rédigé en août 2026, sur la base de la version 1.0 de l'Exam Guide.*

J'ai récemment préparé et réussi l'examen **Claude Certified Architect – Foundations**, après cinq semaines de préparation en parallèle de mon emploi à temps plein. Ce chiffre mérite d'être nuancé : j'utilisais déjà Claude Code depuis près d'un an en environnement d'entreprise, notamment pour configurer des serveurs MCP en production et construire des agents. Avec cette expérience, je pense que deux semaines bien ciblées suffiraient à un praticien expérimenté.

Cette expérience s'est révélée plus transférable que je ne le pensais. Les scénarios de l'examen sont fondamentalement des problèmes d'arbitrage : fiabilité contre simplicité, isolation contre contexte partagé, capacité contre rayon d'impact (*blast radius*), latence contre coût. Le raisonnement sous-jacent est familier à quiconque a conçu des systèmes logiciels non triviaux.

Mon principal constat : ce n'est pas avant tout un examen de connaissance de l'API Claude ou de prompt engineering. C'est un examen d'architecture et de prise de décision, construit autour de scénarios de production réalistes.

## L'examen en un coup d'œil

Avant de parler de préparation, voici le format tel que publié dans l'Exam Guide officiel.

- **Questions** : 60, à choix multiple, une seule bonne réponse
- **Durée** : 120 minutes
- **Scénarios** : 4 tirés d'un pool de 6, chacun servant de contexte narratif à environ un quart des questions
- **Domaines** : 5, pondérés de façon inégale
- **Score de réussite** : 720 sur une échelle de 100 à 1 000 — un score pondéré, pas un pourcentage de bonnes réponses
- **Passage** : Pearson VUE, en ligne sous surveillance ou en centre d'examen
- **Tarif** : 125 $
- **Validité du certificat** : 12 mois
- **Profil visé** : environ six mois d'expérience pratique avec les API Claude, l'Agent SDK, Claude Code et MCP — pas seulement quelqu'un ayant suivi des tutoriels

Un point à vérifier avant d'investir plusieurs semaines de préparation : l'éligibilité. En août 2026, le programme de certification Claude est ouvert aux organisations membres du Claude Partner Network, et réussir l'examen compte dans le statut de l'organisation au sein du réseau — l'inscription se fait via votre employeur s'il en fait partie, avec votre adresse e-mail professionnelle.

La page de certification, elle, est publique : seule l'inscription est verrouillée. N'importe qui peut consulter la pondération des domaines, le format, et surtout télécharger l'Exam Guide depuis cette page. Si votre organisation ne fait pas partie du réseau, l'examen reste hors de portée pour l'instant — mais la préparation décrite ci-dessous, elle, ne l'est pas.

Les cinq domaines notés, dans l'ordre du guide :

1. Agentic Architecture & Orchestration — 27 %
2. Tool Design & MCP Integration — 18 %
3. Claude Code Configuration & Workflows — 20 %
4. Prompt Engineering & Structured Output — 20 %
5. Context Management & Reliability — 15 %

L'important n'est pas de mémoriser ces chiffres, mais de comprendre comment ces domaines interagissent dans un scénario de production — et de remarquer qu'un seul domaine représente plus d'un quart de l'examen.

## Que teste réellement la certification ?

Une idée reçue avant de commencer la préparation est de penser que cet examen porte principalement sur la connaissance de l'API Claude ou de Claude Code. Ce n'est pas le cas.

Les questions s'appuient sur des situations clients réalistes — agents de support client, systèmes de recherche multi-agents, workflows de développement avec Claude Code, CI/CD, productivité des développeurs, extraction de données structurées — et on attend de vous des décisions d'architecture, de configuration et d'arbitrage de production. La formulation reflète cela : on ne vous demande pas si une fonctionnalité existe, on vous donne une situation, un ensemble de contraintes et plusieurs options techniquement plausibles, puis on vous demande laquelle correspond le mieux.

Cette nuance compte, car plusieurs réponses peuvent fonctionner dans la réalité. La bonne réponse est celle qui traite le problème réel tout en respectant toutes les contraintes du scénario. Le guide officiel est explicite sur ce point : les mauvaises options sont rédigées pour paraître raisonnables à quelqu'un dont les connaissances ou l'expérience sont incomplètes. Par exemple, on ne vous demandera pas simplement :

> « Que fait la fonctionnalité X ? »

Vous êtes bien plus susceptible de rencontrer une question du type :

> « Votre agent en production se comporte mal dans ces conditions. Quel changement architectural résoudrait le problème le plus efficacement ? »

## Les pièges architecturaux que j'ai sous-estimés

C'est la partie à laquelle je consacrerais plus de temps si je devais recommencer.

**Le batching n'est pas une optimisation de coût.** J'avais classé la Message Batches API dans la case « moins cher » et n'y avais plus touché. Ce qui compte réellement, c'est de savoir si le workflow est bloquant : une vérification pré-merge que des développeurs attendent a besoin d'une latence prévisible, un rapport de dette technique généré la nuit n'en a pas besoin. Dès que j'ai commencé à lire les scénarios en cherchant le caractère bloquant plutôt que le coût, cette famille de questions a cessé d'être ambiguë.

**Ressources MCP contre outils (tools).** Dans les serveurs que j'avais construits, presque tout était exposé comme un outil ; les outils sont familiers et faciles à invoquer. Mais quand un agent doit appeler un outil simplement pour découvrir quel contenu existe, chaque tour de conversation sert à explorer, pas à agir. Le contenu que l'agent doit simplement lire relève d'une ressource ; les actions qui modifient un état relèvent d'un outil. Appliquer cette distinction à mon travail a nettement réduit le bruit d'exploration — c'est ainsi que je sais qu'elle n'est pas que théorique.

**Le contexte est un budget, pas un conteneur.** Mon réflexe sur les tâches longues était de tout garder dans une seule conversation et de la compacter quand elle devenait lourde, ou de systématiquement repartir sur une nouvelle session. La discipline la plus efficace consiste à dépenser le contexte délibérément : élaguer les sorties d'outils verbeuses, conserver des faits structurés plutôt que la transcription brute, isoler le contexte des sous-agents pour que l'exploration ne touche jamais le fil principal, et persister l'état quand le travail dépasse les limites du contexte.

**La conception du schéma l'emporte sur les instructions de prompt.** Le réflexe habituel pour améliorer un prompt est d'ajouter une phrase de plus : « fais attention, n'invente pas, dis quand tu ne sais pas ». Concevoir le schéma de sortie pour que l'incertitude soit représentable est plus efficace : champs nullables, enums adaptés, un cas « inconnu » explicite. Un modèle qui dispose d'un moyen valide de dire que l'information manque cesse de combler le vide avec quelque chose de plausible.

**Garantie déterministe contre conformité probabiliste.** C'est le schéma sous-jacent à tous les autres. Si une règle métier doit toujours être respectée, la réponse est un hook ou un prérequis programmatique — pas une instruction de plus dans le system prompt. Toute option qui demande au modèle d'être fiable sur un point que le système pourrait simplement imposer est en général l'option à éliminer.

C'est là que l'expérience de production m'a le plus aidé — et c'est aussi là qu'elle peut induire en erreur. L'architecture que vous exploitez déjà peut très bien fonctionner ; l'examen demande quel schéma est le plus approprié compte tenu des contraintes données. Par exemple, quand un client demande explicitement un agent humain pour une simple réinitialisation de mot de passe, mon réflexe de production — optimisé pour la déflection de tickets — était d'essayer de résoudre la demande de façon autonome avant d'escalader. Anthropic exige au contraire de respecter immédiatement toute demande explicite d'agent humain, sans tentative d'investigation préalable. Les habitudes d'économie propres à l'entreprise peuvent facilement vous orienter vers la mauvaise réponse.

## Ce que j'ai réellement utilisé pour me préparer

Ma préparation a été délibérément pratique.

**Apprendre :** j'ai suivi les cours gratuits d'Anthropic Academy, en particulier *Building with the Claude API* et *Claude Code in Action*. Le premier couvre l'usage des outils, la sortie structurée, MCP et les workflows d'agents ; le second se concentre sur la configuration, le Plan Mode, les skills, les hooks, l'automatisation et les workflows de longue durée. Je ne me suis pas contenté de les regarder.

**Construire :** chaque concept reproduit en code plutôt que simplement visionné — boucles d'agents, configuration de Claude Code, outils MCP, un pipeline d'extraction structurée, un petit système multi-agents. L'examen récompense la compréhension opérationnelle plus que la reconnaissance, et l'écart entre les deux n'apparaît que face à un scénario.

**Schématiser :** une tablette toujours à portée de main, pour redessiner chaque pattern avant de considérer l'avoir compris. Dessiner un orchestrateur qui délègue à trois workers, puis la même tâche sous forme de pipeline parallèle, rend la différence évidente : la topologie est presque identique, seul le moment où les sous-tâches sont décidées les distingue.

Une chose que je changerais, en revanche, c'est l'ordre. J'ai commencé par les cours et n'ai vraiment étudié l'Exam Guide qu'après. Je ferais l'inverse : **lire d'abord l'Exam Guide.**

L'Exam Guide est la carte : domaines, objectifs, scénarios et exemples de questions. Lisez-le avant de commencer les cours, puis transformez chaque objectif en checklist simple :

Est-ce que je peux l'expliquer ? Est-ce que je peux l'implémenter ? Est-ce que je peux reconnaître le mauvais choix architectural dans un scénario ?

Chaque « non » devient un sujet d'étude. Cela a changé ma façon d'utiliser les cours : au lieu de me demander si un concept était important, je savais exactement quel objectif il m'aidait à couvrir.

## Ma boucle de préparation

Mon processus réel est devenu :

**Exam Guide → Cours → Pratique → Examen blanc → Analyse des erreurs → Documentation → Recommencer**

J'ai généré des examens blancs avec Gemini Pro, en utilisant l'Exam Guide comme source de vérité. Il n'existe pas d'examen blanc officiel complet, même si le guide inclut des exemples de questions commentées. L'idée n'était pas de générer des questions au hasard : je voulais des questions qui reproduisent le style de raisonnement de l'examen — scénarios réalistes, contraintes explicites, quatre options plausibles et une seule bonne réponse.

```
Using only the exam objectives provided below, generate one
scenario-based multiple-choice question.

Requirements:
- Create a realistic production architecture scenario
- Include explicit constraints such as latency, cost,
  reliability, security, or context limits
- Provide 4 technically plausible options
- Have exactly one best answer given the stated constraints
- After I answer, explain why each option is correct or incorrect
- For every incorrect option, identify the constraint or
  architectural principle it violates

Do not use information outside the provided objectives.
Do not reproduce real exam questions.
```

Cela a fini par être l'une des parties les plus utiles de ma préparation, car je pouvais générer de nouvelles questions dès que je découvrais un point faible.

Mais une règle importante : les réponses générées sont un entraînement, pas une vérité. Ma hiérarchie des sources était l'Exam Guide, puis la documentation d'Anthropic, puis les cours officiels, puis ma propre expérience, et seulement ensuite le contenu généré. Dès qu'une explication d'examen blanc me semblait douteuse, je remontais cette hiérarchie.

## Avis sur les examens blancs tiers

Côté ressources externes, j'ai fait un examen blanc de Frank Kane sur Udemy — *Anthropic Claude Certified Architect – Full Practice Exams* — utile pour exercer ce raisonnement avec un style de questions différent.

En dehors de ça, je n'ai rien trouvé de vraiment utile en ligne à la date de juillet 2026. Si vous connaissez une bonne ressource, n'hésitez pas à la partager en commentaire.

Quelle que soit la ressource utilisée, ne cherchez pas à maximiser le nombre d'examens blancs faits. La valeur vient de ce qui se passe après la question : pour chaque erreur, retournez au cours ou à la documentation, identifiez précisément ce que vous aviez mal compris, puis trouvez une autre question ciblant le même concept.

## Comment j'ai abordé les questions difficiles

Ma stratégie par défaut était l'élimination. Quand je ne connaissais pas la réponse immédiatement, j'arrêtais de me demander « quelle option est correcte ? » pour me demander « quelles options puis-je éliminer ? »

Pour chaque choix, je vérifiais :

- Résout-il le problème réel ?
- Respecte-t-il toutes les contraintes explicites ?
- Introduit-il une infrastructure que le scénario ne requiert pas ?
- Donne-t-il à un agent plus d'outils ou de permissions que nécessaire ?
- Repose-t-il sur un comportement probabiliste là où une application déterministe est requise ?

En général, deux options deviennent faciles à éliminer, et il reste un arbitrage architectural bien plus restreint. Mon modèle mental pendant l'examen est devenu :

**Scénario → Objectif → Contraintes → Éliminer → Comparer les arbitrages → Choisir**

Le piège consiste à choisir une réponse parce qu'elle fonctionnerait. Fonctionner ne suffit pas : la question porte sur la conception que le scénario exige réellement.

## Gestion du temps

La limite de 120 minutes donne en moyenne deux minutes par question, mais je ne traiterais pas ces deux minutes comme un objectif. Certaines questions se répondent en quelques secondes dès qu'on maîtrise le concept sous-jacent ; d'autres demandent plusieurs minutes parce qu'il faut lire le scénario attentivement et comparer des arbitrages architecturaux.

J'ai donc avancé rapidement sur les questions évidentes pour préserver ma capacité mentale pour les scénarios longs. Une question lente ne signifie pas que vous vous en sortez mal — cela signifie généralement qu'elle teste votre jugement plutôt que votre mémoire.

## Les erreurs que j'éviterais

- Commencer par les cours plutôt que par l'Exam Guide.
- Regarder les leçons au lieu de les reproduire en code, selon votre niveau d'expérience avec Claude.
- Faire confiance à des réponses générées ou tierces sans vérifier la documentation.
- Mémoriser les réponses des examens blancs au lieu de comprendre pourquoi les autres options échouent.
- Supposer que ce que vous exploitez déjà en production est automatiquement le pattern attendu par l'examen.
- Apprendre de nouvelles notions la veille — j'ai utilisé le dernier jour pour relire l'Exam Guide et consolider, pas pour élargir le programme.

## Ce que je retiens de cette certification

Le résultat le plus précieux n'est pas le certificat lui-même. Me préparer m'a obligé à mettre des noms et des limites précis sur des patterns que j'utilisais déjà : orchestration d'agents, isolation des outils, transferts structurés, gestion du contexte, application déterministe et escalade humaine.

Cela a rendu la préparation utile bien au-delà de l'examen. On arrête de penser aux agents comme une collection de prompts et d'outils, et on commence à les considérer comme des systèmes logiciels avec un état, des modes de défaillance, des interfaces, des permissions et des arbitrages architecturaux. C'est, au fond, ce que je pense que la certification évalue :

Pas si vous pouvez faire faire quelque chose à Claude. Mais si vous pouvez concevoir un système autour de Claude qui se comporte correctement quand les contraintes deviennent réelles.

Et cela voyage. L'examen est écrit autour de la stack d'Anthropic, mais l'essentiel de ce qu'il évalue n'est pas spécifique à un fournisseur : quand laisser un modèle décider et quand imposer une règle dans le code, comment répartir une tâche entre agents et quand ne pas le faire, ce qui relève de la fenêtre de contexte et ce qui relève d'un fichier, comment concevoir un schéma capable de représenter ce que le modèle ne sait pas. Ces décisions se ressemblent, que le modèle sous-jacent soit Claude, Gemini ou GPT. Le raisonnement architectural se transfère, et c'est ce qui garde sa valeur quand la stack change sous vos pieds.

---

Bonne chance à tous ceux qui préparent l'examen Claude Certified Architect – Foundations. Et si vous l'avez déjà passé, je serais curieux de savoir quelle partie de la préparation vous a semblé la plus difficile.
