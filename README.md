# ![Immutable Web Apps](icons/favicon-32x32.png) Applications Web Immutables

## ![Introduction](icons/favicon-16x16.png) Introduction

_Applications Web Immutables_ est une méthodologie 'framework-agnostic' (sans framework associé) permettant de construire et de déployer des [single-page applications](https://en.wikipedia.org/wiki/Single-page_application) (applications mono-page) en:

- Minimisant les risques et complixitées relatives aux déploiements.
- Simplifiant and maximisant l'efficacité de la mise en cache.
- Minimisant les besoins en termes de serveurs et d'administration d'environnements 'runtime'.
- Permettant une mise en production continue via des déploiements simples, flexibles et atomiques.

## ![Principes](icons/favicon-16x16.png) Principes

La méthodologie s'appuie sur un principe de __stricte séparation__ :

- De la configuration vis-à-vis du code,
- Des tâches de déploiement vis-à-vis des tâches de 'build',
- Du contenu dynamique vis-à-vis du contenu statique.

## ![Concepts](icons/favicon-16x16.png) Concepts

Les concepts suivants définissent les minima requis pour une __Application Web Immutable__. Ils ne dépendent d'aucun framework ni d'aucune infrastructure.

### Les ressources statiques sont indépendantes de l'environnement d'une application web

Les ressources statiques sont les fichiers (javascript, css, images) générés à la construction une applicaiton web depuis son code source. Elles deviennent _immutables_ quand elles ne dépendent d'aucune spécificité d'environnement et qu'elles sont publiées dans un lieu unique et indépendant.

#### Les ressources statiques ne doivent pas rien contenir qui soit spécifique à un environnement

Tous les frameworks d'applications majeurs ([Angular CLI](https://github.com/angular/angular-cli/wiki/stories-application-environments), [Create React App](https://cli.vuejs.org/guide/mode-and-env.html#using-env-variables-in-client-side-code), [Ember CLI](https://ember-cli.com/user-guide/#Environments), [Vue CLI 3](https://cli.vuejs.org/guide/mode-and-env.html#using-env-variables-in-client-side-code)) recommendent de définir les valeurs d'environnements à la _compilation_. Cette pratique demande que les ressources statiques soient générées pour chaque environnement et générées à nouveau pour chaque changement apporté à un environnement.

_Applications Web Immutables_ référence les _variables_ d'environnements définies globalement de deux façons :

- Directement depuis l'objet `window`
- Depuis un service injecté qui 'wrap' (entoure) les variables d'environnements

```diff
  export class UserService {
    webServiceUrl: String;

    constructor(
        private http: HttpClient
        ) {
-         // Supprimez toute configuration écrite en dur ou inclue durant la compilation
-         this.webServiceUrl = 'https://api.myapp.com'
+         // Utilisez des variables définies de façon globale uniques au déploiement
+         this.webServiceUrl = window.env.API
        }

    getUsers() {
      return this.http.get(`${this.webServiceUrl}/users`);
    }
  }
```

Les valeurs pour les _variables_ d'environnements sont définies dans un `index.html` unique pour chaque environnement.

#### Les ressources statiques doivent être hébergées depuis des emplacements uniques et _indépendants de(s) environnement(s) de l'application web_

Les ressources statiques qui ne contiennent aucune spécificité dû à l'environnement peuvent être construite seulement une fois, publiées dans un emplacement unique, et utilisées par de multiples environnements pour une même application web.

Les ressources statiques partagent les caractéristiques des librairies javascript publiées via CDN (content delivery networks) ([Google Hosted Libraries](https://developers.google.com/speed/libraries/), [cdnjs](https://cdnjs.com/), [jsDelivr](https://www.jsdelivr.com/), [UNPKG](https://unpkg.com/)):

<code>https:/<span></span>/<span style="font-weight: bold">ajax.googleapis.c<span></span>om</span>/ajax/libs/jquery/<span style="font-weight: bold">3.3.1</span>/jquery.min.js</code>

L'emplacement de la librairie jQuery ci-dessus, référencée par d'innombrables applications web, est à la fois indépendante des-dites applications et versionnée de façon unique.

<code>https:/<span></span>/<span style="font-weight: bold">assets.myapp.c<span></span>om</span>/apps/<span style="font-weight: bold">1.0.2</span>/main.js</code>

De même, l'emplacement des fichiers javascript de l'application web est unique et publié dans un espace dédié aux ressources statiques. Le serveur de ressources statiques est un répertoire de versions de l'application web.

#### Configurez les ressources statiques pour de la mise en cache à long terme

Les ressources statiques qui ne contiennent aucune spéficité dûe à l'environnement et qui sont publiées depuis un emplacement unique et permanent peuvent être [mis en cache de quasi-indéfiniment](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#Caching_static_assets):

 `cache-control: public, max-age=31536000, immutable`

### `index.html` est une configuration déployable

Le document HTML d'une application mono-page (souvent `index.html`) n'est ___pas___ statique. Il varie d'environnement en environnement et de déploiement en déploiement. Il est un assemblage de la configuration spécifique à l'environnement et des ressources statiques _immutables_ qui définissent l'application web.

#### `index.html` contient des références pleinement qualifiées aux ressources statiques

```diff
- <!-- ni unique, ni indépendant -->
- <script src="main.js" type="text/javascript"></script>

+ <!-- permanent, réutilisable -->
+ <script src="https://assets.myapp.com/apps/1.0.2/main.js" type="text/javascript"></script>
```

#### `index.html` défini les valeurs des variables d'environnement globales

```html
<script>
env = {
    API: 'https://api.myapp.com',
    GA: 'UA-126263119-1'
}
</script>
```

#### `index.html` ne doit jamais être mis en cache

Afin de permettre à l'application web d'être changée de manière instantannée, `index.html` ne doit [jamais être mis en cache par le navigateur ou un cache public ne pouvant pas être purgé à la demande:](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#Preventing_caching)

`cache-control: no-store`

##### _Exemple_

Le fichier `index.html` d'une majorité d'application web mono-page est un document de taille réduite. En y incluant les références versionnées des ressources statiques de l'application web, et en y définissant les variables d'environnement, il devient alors un manifeste de déploiement:

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>My App</title>
    <base href="/">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
    <body>

        <!-- variables d'environnement -->
        <script>
        env = {
            API: 'https://api.myapp.com',
            GA: 'UA-126263119-1'
        }
        </script>

        <!-- ancre pour l'application -->
        <app-root></app-root>

        <!-- ressources statiques pleinement qualifiées -->
        <script src="https://assets.myapp.com/apps/1.0.2/main.js" type="text/javascript"></script>

    </body>
</html>
```

## ![Workflow](icons/favicon-16x16.png) Workflow

_Applications Web Immutables_ sépare ses tâches de déploiement et de construction dans deux workflows distincts.

### Construction

Le code d'une _Application Web Immutable_ à la responsabilité de construire des ressources statiques et de les publier depuis un serveur web statique. Chaque état de la 'codebase' n'a pas obligation d'être publié, mais chacun de ces états ne devrait jamais être publié plus d'une fois.

Généralement, son code est soumis à un contrôle de version via répertoire, avec un système d'intégration continue capable de construire, versionner, et publié des ressources statiques vers un server web statique.

_Exemple pratique :_

Un projet __Angular__ est hebergé dans un repo __GitHub__. Le repo est intégré avec __TravisCI__ afin de construire et de versionner les ressources poussées sur la branche master. Les ressources versionnées sont publiées dans un emplacement unique dans un bucket __AWS S3__.

### Déploiement

Les `index.html` sont gérés indépendamment de la 'codebase' et servent de manifeste pour chaque environnement. Ils doivent être considérés comme des fichiers de configuration et gérés comme tel. De plus, il est nécessaire de créer un mechanisme pour modifier et remplacer les `index.html` dans chaque environnement. Changer un `index.html` est en soit un déploiement.

_Exemple pratique :_

Un ensemble de fichiers `index.html`, un pour chaque environnement, hébergé dans un repo __Github__. Le repo est intégré avec __TravisCI__ afin de publier chaque `index.html` modifié vers __AWS S3 bucket__. Chaque environnement possède son propre `index.html` et son propre bucket S3.

### Infrastructure

![Infrastructure](assets/infrastructure.png)

L'infrastructure pour supporter une __App Web Immutable__ est composée de trois parties :

- __Serveur d'Application Web__: Un serveur statique pour hébergé l'environnement de l'application web en servant l'`index.html`.
- __Serveur de Ressources Statiques__: Un serveur web statique hébergeant les ressources statiques immutables.
- __API__: Un ou plusieurs endpoint exposés afin d'intéragir avec le backend de l'application web.

Les deux serveurs statiques ont des comportements différents :

| | Serveur d'Application Web | Serveur de Ressources Statiques |
| --- | --- | --- |
| Contenu | `index.html` | Ressources Statiques (`*.js`,`*.css`, images, etc)|
| Routage | toujours `index.html` | le fichier disponible à l'Url demandée |
| Contrôle du cache | `no store` | `cache-control: public, max-age=31536000, immutable` |
| Instances | Une par environnement de l'application web | Une par application web

## ![Benefices](icons/favicon-16x16.png) Bénéfices

### Déploiement améliorés

Construire des ressources statiques est un processus compliqué qui implique souvent de :

- résoudre des dépendances,
- downloader des librairies,
- transpiler
- minifier,
- bundler,
- uglifier, diviser le code, de faire du tree-shaking, autoprefixer... la liste est longue...

Ces processus prennent du temps, s'appuie sur les dépendences externes, et se comportent souvent de manière non-déterministe. Ce ne sont pas des processus qui devraient être conclus par la publication des ressources générées en production sans validation. Même l'action de publier de plusieurs ressources statiques lourdes est un processus qui peut être interrompu en laissant l'environnement de l'application web dans un état corrompu.

Les __Applications Web Immutables__ sont générées une seule fois et publiées une seule fois à un emplacement permanent. Ce processus occure en avance du déploiement 'live'. Elles peuvent être validées dans un environnement de test, puis promuent vers l'environnement de production sans avoir besoin d'être regénérées, limitant significativement les risques.

### Déploiement live atomique

Le déploiement live d'un _Application Web Immutable_ se réduit à l'acte de publier un seul fichier `index.html`. Le déploiement est instantanné, et toutes les ressources sont disponibles immédiatement sans risque que le cache soit corrompu au moment du déploiement.

Les retours en arrière sont aussi risqués que les déploiements, si ce n'est plus risqués encore. Pour les _Applications Web Immutables_, les bénéfices du déploiement s'appliquent au 'rollback'. Notamment, dans le cas d'un rollback, la plupart des navigateurs auront toujours à disposition les ressources précédemment mises en cache.

Dans le cas improbable où un navigateur essayerait de charger une version périmée de l'`index.html`, toutes les assets de la version précédentes seront toujours disponibles et non corrompues.

### Mise en cache simplifiée

Manager les headers `cache-control` peut être intimidant, spécialement quand l'infrastructure de l'application web utilises des caches publiques comme ceux utilisés par les CDNs. Les deux concepts de gestion de cache les plus simples sont : "Toujours mettre en cache" et "Ne jamais mettre en cache". _Application Web Immutable_ embrasse complètement ces concepts en séparant le code "Toujours mis en cache" de la configuration "Jamais mise en cache".

### Routage simplifié

Les frameworks leaders actuels du marché ne séparent pas les ressources statiques du `index.html` dans leurs [recommendations de déploiement](https://angular.io/guide/deployment#server-configuration). À la place, ils recommandent d'ajouter des règles de routage au serveur web qui retourne l'`index.html` pour toutes les routes qui dirigent pas vers un fichier physique. L'implémentation de ces règles de routage peuvent différer d'un serveur web à un autre et des erreurs peuvent souvent résulter en des routes menant aux mauvaises ressources.

Séparer l'hébergement de l'`index.html` et des ressources statiques elimine ce risque. Le serveur de ressources statiques sert toujours un fichier physique représenté par l'url et serveur d'application web sert toujours un `index.html`, quelque soit l'url.

## ![Background](icons/favicon-16x16.png) Background

La méthodologie _Applications Web Immutables_ est construite sur les bases de plusieurs tendances dans le développement d'application web :

- __Frameworks d'Application Moderne:__ Angular, React, Vue, et Ember ont permi à des nombreuses équipes de créer des applications mono-page complexes statiques. Des outils comme webpack ont amélioré la capacité à créer, optimiser et gérer les artefacts de 'build'.

- __DevOps:__ La culture DevOps a permi aux développeurs d'applications web de décomposer et réévaluer leur infrastructure afin de mieux servir les besoins de leurs applications web.

- __Maturation des modèles d'applications et des pratiques:__ Les applications et services Backend convergent vers un ensemble de 'best-practices' supportant la portabilité, la flexibilité et la haute-disponibilité. Cette tendance a grandement augmentée le nombre d'outils et de services disponibles, surtout en ce qui concerne les containers et leur orchestration. Beaucoup de ces pratiques commencent maintenant à s'appliquer aux applications web statiques mono-page.

## ![Influences](icons/favicon-16x16.png) Influences

 - [__The Twelve-Factor App__](https://12factor.net/): Cette méthodologie de construction d'applications web est basée sur une séparation claire des responsabilités afin d'assurer portabilité et robustesse. _Application Web Immutable_ sépare beaucoup des même responsabilités afin d'atteindre les même objectifs.

 - [__JAMstack__](https://jamstack.org/): La méthodologie _Application Web Immutable_ s'aligne complètement avec les [best practices](https://jamstack.org/best-practices/) du JAMstack.

## ![Projects](icons/favicon-16x16.png) Projects

- [__ng-immutable-example__](https://github.com/ImmutableWebApps/ng-immutable-example): Converti un projet généré par [Angular CLI](https://cli.angular.io/) en une Application Web Immutable Web.

- [__react-immutable-example__](https://github.com/ImmutableWebApps/react-immutable-example): Converti un projet généré par [Create React App](https://facebook.github.io/create-react-app/) en une Application Web Immutable Web.

- [__unpkg-immutable-example__](https://github.com/ImmutableWebApps/unpkg-immutable-example): Un exemple d'Application Web Immutable hébergé sur [npm](https://www.npmjs.com/), [UNPKG](https://unpkg.com/#/), et [GitHub Pages](https://pages.github.com/).

- [__aws-lambda-edge-example__](https://github.com/ImmutableWebApps/aws-lambda-edge-example): Déploit une Application Web Immutable [AWS Lambda@Edge](https://aws.amazon.com/lambda/edge/) avec [Serverless](https://serverless.com/).

## ![Talks & Articles](icons/favicon-16x16.png) Talks & Articles

- [_"Single-page App Deployments"_](https://www.meetup.com/nh-js-manchester/events/255671618/), Gene Connolly, [NH.js](https://www.meetup.com/nh-js-manchester/), 14 Novembre 2018

- [_"Risk-free Deployments with Immutable Web Apps"_](https://underthehood.meltwater.com/blog/2018/12/03/risk-free-deployments-with-immutable-web-apps/), Gene Connolly, [underthehood.meltwater.com](https://underthehood.meltwater.com), 3 Décembre 2018

## ![Travaux Futurs](icons/favicon-16x16.png) Travaux Futurs

_Application Web Immutable_ n'est pas activement supporté par un framework, outil ou service à l'heure actuelle. La majorité des applications mono-page devraient pouvoir adopter la méthodologie _Application Web Immutable_ en apportant des changements mineurs à leur codebase, processus de 'build' et infrastructure. Cependant, avant que le support de cette méthodologie ne se généralise, les applications web utilisant des techniques de [division de code](https://webpack.js.org/guides/code-splitting/) avancées risquent de rencontrer des complications et inadéquations lors de son adoption.

Atteindre une adoption générale demandera :

- __La création de tutoriels, documentations et exemples__ sur la façon de créer des _Applications Web Immutables_ utilisant les frameworks et outils leaders du marché,

- __De proposer des changements aux frameworks et outils existant__ afin de supporter les _Applications Web Immutables_.

- __De promouvoir et d'éduquer__ sur les _Applications Web Immutables_ à travers des articles de blog et des présentations.

## ![Acknowledgements](icons/favicon-16x16.png) Acknowledgements

_Applications Web Immutables_ fut développé chez [Meltwater](https://www.meltwater.com/) à partir de recherches et de l'expérience gagnée à travers la création d'applications web. Apprenez-en plus sur les choses que nous produisons sur notre blog: [underthehood.meltwater.com](https://underthehood.meltwater.com/). 

immutablewebapps@gmail.com
[@immutablewebapp](https://twitter.com/ImmutableWebApp)