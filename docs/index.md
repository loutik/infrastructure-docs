# Bienvenue 👋

![Bannière LoutikCloud](https://raw.githubusercontent.com/loutik/design-assets/main/loutikcloud/banniere_loutikcloud.png)

## 📖 Introduction

Bienvenue dans la documentation technique du projet **LoutikCLOUD** ! 👋

### ***Mais qu'est-ce que Loutik au juste ?***

Loutik est un projet personnel que j'ai créé pour nourrir ma passion pour l'ingénierie système. Mon objectif est de simuler un véritable environnement de production pour héberger des outils open source, tout en respectant l'état de l'art de l'industrie (SRE, FinOps, GreenOps). C'est mon laboratoire d'expérimentation pour "casser proprement" et apprendre à réparer.

### ***Qui suis-je ?***

<div style="display: flex; flex-wrap: wrap; gap: 1rem; align-items: flex-start; justify-content: flex-start;">
  
  <!-- Conteneur Image + Légende -->
  <div style="display: flex; flex-direction: column; align-items: center; margin-left: auto; margin-right: auto; width: fit-content;">
    <img src="assets/photo_louismedo.jpg" alt="Photo de Louis MEDO" style="width: 200px; height: 266px; object-fit: cover; border-radius: 4px;">
    <span style="margin-top: 0.5rem; font-size: 0.7rem; color: #555; text-align: center;">Photo de Louis MEDO</span>
  </div>

  <!-- Bloc Texte -->
  <div style="flex: 1 1 300px; min-width: 250px;">
    <p>
      Je m'appelle Louis MEDO, et je suis un passionné d'informatique. Ma nature curieuse m'a toujours poussé à comprendre le fonctionnement des choses, des frigos aux ordinateurs. J'ai commencé par le hardware : monter des PC, comprendre le stockage... Puis est venu le déclic au collège avec la découverte d'un logiciel de vie scolaire bien connu, Pronote. Ma curiosité a été piquée au vif : comment cela fonctionnait-il ? En cherchant, j'ai découvert qu'on pouvait l'auto-héberger. Ma vocation de <em>sysadmin</em> était née.
    </p>
    <p>
      LoutikCLOUD est l'aboutissement de ce parcours. C'est ma plateforme pour concevoir une infrastructure d'entreprise, de la documentation <em>as Code</em> au déploiement des services. Outre la technique, je souhaite également héberger des projets open source pour permettre au plus grand nombre de tester ces outils gratuitement.
    </p>
  </div>

</div>

### ***Pourquoi faire une documentation ?***

Pour moi, la documentation est la fondation d'une infrastructure. Sans organisation, un système s'effondrera à la première défaillance de production, avec des conséquences lourdes (j'en ai fait l'expérience, croyez-moi !). C'est pourquoi j'applique la méthode "Docs-as-Code". Écrire ces guides me permet de valider mes acquis, de partager mes découvertes, et de garantir la reproductibilité de mon environnement.

## 🧭 Plan de Navigation

J'ai structuré la documentation en suivant l'organisation de l'infrastructure elle-même (Component-Driven). C'est intuitif : si vous cherchez le réseau, c'est dans *Edge* ; si vous cherchez une application, c'est dans *Services*. Le framework Diátaxis est quant à lui appliqué à l'intérieur des dossiers complexes pour séparer la pratique de la théorie.

Explorez les sections principales :

<!-- Conteneur principal : passé en flex-direction column pour empiler les cartes -->
<div style="display: flex; flex-direction: column; gap: 1.5rem; margin-top: 2rem; width: 100%;">

  <!-- 1. Architecture -->
  <a href="01-architecture/" style="text-decoration: none; color: inherit; display: block; width: 100%;">
    <div style="display: flex; align-items: flex-start; padding: 1.5rem; background-color: #ffffff; border: 1px solid rgba(13, 27, 42, 0.08); border-radius: 12px; transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease; height: 100%; font-size: 1rem; line-height: 1.5; width: 100%; box-sizing: border-box;" 
         onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 20px rgba(0,0,0,0.05)'; this.style.borderColor='#00897B';" 
         onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='none'; this.style.borderColor='rgba(13, 27, 42, 0.08)';">
      
      <div style="font-size: 2.2rem; margin-right: 1.2rem; margin-top: 0.2rem; color: #00897B; flex-shrink: 0;">🏗️</div>
      
      <div style="flex: 1;">
        <h3 style="margin: 0 0 0.5rem 0; font-size: 1rem; font-weight: 700; color: #0d1b2a; line-height: 1.2;">1. Architecture</h3>
        <p style="margin: 0; font-size: 0.75rem; color: #1b263b; line-height: 1.6;">Conception et gouvernance : Registre des décisions d'architecture (ADR), standards de nommage, et retours d'expérience (Post-mortems) analytiques sur les incidents de production.</p>
      </div>
    </div>
  </a>

  <!-- 2. Infrastructure Core -->
  <a href="02-infrastructure-core/" style="text-decoration: none; color: inherit; display: block; width: 100%;">
    <div style="display: flex; align-items: flex-start; padding: 1.5rem; background-color: #ffffff; border: 1px solid rgba(13, 27, 42, 0.08); border-radius: 12px; transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease; height: 100%; font-size: 1rem; line-height: 1.5; width: 100%; box-sizing: border-box;" 
         onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 20px rgba(0,0,0,0.05)'; this.style.borderColor='#00897B';" 
         onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='none'; this.style.borderColor='rgba(13, 27, 42, 0.08)';">
      
      <div style="font-size: 2.2rem; margin-right: 1.2rem; margin-top: 0.2rem; color: #00897B; flex-shrink: 0;">🧱</div>
      
      <div style="flex: 1;">
        <h3 style="margin: 0 0 0.5rem 0; font-size: 1rem; font-weight: 700; color: #0d1b2a; line-height: 1.2;">2. Infrastructure Core</h3>
        <p style="margin: 0; font-size: 0.75rem; color: #1b263b; line-height: 1.6;">Fondations de l'infrastructure : Plateformes de virtualisation (Proxmox), topologie réseau LAN, gestion du stockage unifié (SAN/NAS) et configurations du socle technique.</p>
      </div>
    </div>
  </a>

  <!-- 3. Edge -->
  <a href="03-edge/" style="text-decoration: none; color: inherit; display: block; width: 100%;">
    <div style="display: flex; align-items: flex-start; padding: 1.5rem; background-color: #ffffff; border: 1px solid rgba(13, 27, 42, 0.08); border-radius: 12px; transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease; height: 100%; font-size: 1rem; line-height: 1.5; width: 100%; box-sizing: border-box;" 
         onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 20px rgba(0,0,0,0.05)'; this.style.borderColor='#00897B';" 
         onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='none'; this.style.borderColor='rgba(13, 27, 42, 0.08)';">
      
      <div style="font-size: 2.2rem; margin-right: 1.2rem; margin-top: 0.2rem; color: #00897B; flex-shrink: 0;">🌐</div>
      
      <div style="flex: 1;">
        <h3 style="margin: 0 0 0.5rem 0; font-size: 1rem; font-weight: 700; color: #0d1b2a; line-height: 1.2;">3. Edge</h3>
        <p style="margin: 0; font-size: 0.75rem; color: #1b263b; line-height: 1.6;">Périmètre et exposition : Sécurisation périmétrique (Firewalling OPNsense), routage des flux externes (Reverse Proxy), gestion DNS et répartition de charge (Load Balancing).</p>
      </div>
    </div>
  </a>

  <!-- 4. Sécurité -->
  <a href="04-securite/" style="text-decoration: none; color: inherit; display: block; width: 100%;">
    <div style="display: flex; align-items: flex-start; padding: 1.5rem; background-color: #ffffff; border: 1px solid rgba(13, 27, 42, 0.08); border-radius: 12px; transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease; height: 100%; font-size: 1rem; line-height: 1.5; width: 100%; box-sizing: border-box;" 
         onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 20px rgba(0,0,0,0.05)'; this.style.borderColor='#00897B';" 
         onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='none'; this.style.borderColor='rgba(13, 27, 42, 0.08)';">
      
      <div style="font-size: 2.2rem; margin-right: 1.2rem; margin-top: 0.2rem; color: #00897B; flex-shrink: 0;">🛡️</div>
      
      <div style="flex: 1;">
        <h3 style="margin: 0 0 0.5rem 0; font-size: 1rem; font-weight: 700; color: #0d1b2a; line-height: 1.2;">4. Sécurité</h3>
        <p style="margin: 0; font-size: 0.75rem; color: #1b263b; line-height: 1.6;">Surveillance et protection : Centralisation et analyse des événements de sécurité (SIEM) et filtrage trafic applicatif (WAF).</p>
      </div>
    </div>
  </a>

  <!-- 5. Services -->
  <a href="05-services/" style="text-decoration: none; color: inherit; display: block; width: 100%;">
    <div style="display: flex; align-items: flex-start; padding: 1.5rem; background-color: #ffffff; border: 1px solid rgba(13, 27, 42, 0.08); border-radius: 12px; transition: transform 0.2s ease, box-shadow 0.2s ease, border-color 0.2s ease; height: 100%; font-size: 1rem; line-height: 1.5; width: 100%; box-sizing: border-box;" 
         onmouseover="this.style.transform='translateY(-4px)'; this.style.boxShadow='0 10px 20px rgba(0,0,0,0.05)'; this.style.borderColor='#00897B';" 
         onmouseout="this.style.transform='translateY(0)'; this.style.boxShadow='none'; this.style.borderColor='rgba(13, 27, 42, 0.08)';">
      
      <div style="font-size: 2.2rem; margin-right: 1.2rem; margin-top: 0.2rem; color: #00897B; flex-shrink: 0;">🚀</div>
      
      <div style="flex: 1;">
        <h3 style="margin: 0 0 0.5rem 0; font-size: 1rem; font-weight: 700; color: #0d1b2a; line-height: 1.2;">5. Services</h3>
        <p style="margin: 0; font-size: 0.75rem; color: #1b263b; line-height: 1.6;">Catalogue applicatif et opérations : Documentation des applications hébergées (Supervision, Cloud privé, etc.), procédures de déploiement et Runbooks opérationnels d'intervention.</p>
      </div>
    </div>
  </a>

</div>