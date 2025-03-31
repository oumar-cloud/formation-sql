DROP DATABASE IF EXISTS my_crm;
CREATE DATABASE my_crm CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE my_crm;


CREATE TABLE client (
 id INT NOT NULL AUTO_INCREMENT,
 nom VARCHAR(255) NOT NULL,
 CONSTRAINT pk_client PRIMARY KEY (id)
)ENGINE=INNODB;

CREATE TABLE projet(
 id INT NOT NULL AUTO_INCREMENT,
 client_id INT NOT NULL,
 nom VARCHAR(255) NOT NULL,
 CONSTRAINT pk_projet PRIMARY KEY (id)
)ENGINE=INNODB;

CREATE TABLE devis(
 id INT NOT NULL AUTO_INCREMENT,
 version INT NOT NULL,
 reference VARCHAR(10) NOT NULL,
 prix FLOAT NOT NULL,
 projet_id INT NOT NULL,
 CONSTRAINT pk_devis PRIMARY KEY (id)
)ENGINE=INNODB;

CREATE TABLE facture(
 id INT NOT NULL AUTO_INCREMENT,
 reference VARCHAR(10) NOT NULL,
 info VARCHAR(255) NOT NULL,
 total FLOAT NOT NULL,
 date_crea DATE NOT NULL,
 date_paiement DATE NULL,
 devis_id INT NOT NULL,
 CONSTRAINT pk_facture PRIMARY KEY (id)
)ENGINE=INNODB;

-- fk_client dans table projet
ALTER TABLE projet 
ADD CONSTRAINT fk_client 
FOREIGN KEY projet(client_id) 
REFERENCES client(id);

-- fk_projet dans table devis
ALTER TABLE devis 
ADD CONSTRAINT fk_projets 
FOREIGN KEY devis(projet_id) 
REFERENCES projet(id);

-- fk_devis dans table facture
ALTER TABLE facture 
ADD CONSTRAINT fk_devis 
FOREIGN KEY facture(devis_id) 
REFERENCES devis(id);

###############################
# Ajouter les données  
###############################

INSERT INTO client (nom) VALUES 
	('Mairie de Rennes'),
	('Neo Soft'),
	('Sopra'),
	('Accenture'),
	('Amazon');
INSERT INTO projet (nom, client_id) VALUES
	('Création de site internet', 1),
	('Logiciel CRM', 2),
	('Logiciel de devis', 3),
	('Site internet e-commerce', 4),
	('Logiciel ERP', 2),
	('Logiciel gestion de stock',5);

INSERT INTO devis (version,reference, prix, projet_id) VALUES
	('1','DEV2100A', 3000, 1),
	('2','DEV2100B', 5000, 1),
	('1','DEV2100C', 5000, 2),
	('1','DEV2100D', 3000, 3),
	('1','DEV2100E', 5000, 4),
	('1','DEV2100F', 2000, 5),
	('1','DEV2100G', 1000, 6);

INSERT INTO facture (reference,info,total,devis_id,date_crea,date_paiement)	
    VALUES
	('FA001', 'site internet partie 1', 1500, 1, '2023-09-01','2023-10-01'),
	('FA002', 'site internet partie 2', 1500, 1, '2023-09-20',null),
	('FA003', 'logiciel CRM', 5000, 3, '2024-02-01',null),
	('FA004', 'logiciel devis', 3000, 4, '2024-03-03','2024-04-03'),
	('FA005', 'site ecommerce', 5000, 5, '2023-03-01',null),
	('FA006', 'logiciel ERP', 2000, 6, '2023-03-01',null);
    ###############################
# Afficher toutes les factures avec le nom des clients
###############################
SELECT facture.reference, client.nom, facture.info, facture.total, facture.date_crea, facture.date_paiement
FROM facture
JOIN devis ON facture.devis_id = devis.id
JOIN projet ON devis.projet_id = projet.id
JOIN client ON projet.client_id = client.id;
  ###############################
# Afficher le nombre de factures par client
###############################
SELECT 
    client.nom AS client,
    COUNT(facture.id) 
FROM client
LEFT JOIN projet ON client.id = projet.client_id
LEFT JOIN devis ON projet.id = devis.projet_id
LEFT JOIN facture ON devis.id = facture.devis_id
GROUP BY client.nom;
 ###############################
# Afficher le chiffre d'affaire par client
###############################
SELECT 
    client.nom AS client,
    SUM(facture.total) AS ca_par_client
FROM client
LEFT JOIN projet ON client.id = projet.client_id
LEFT JOIN devis ON projet.id = devis.projet_id
LEFT JOIN facture ON devis.id = facture.devis_id
GROUP BY client.nom;


use my_crm;
#Le ca_total
SELECT SUM(total) AS ca_total FROM facture;
#Afficher la somme des factures en attente de paiement
SELECT SUM(total) AS total_factures
FROM facture
WHERE date_paiement IS NULL;

use my_crm;
#Afficher les factures en retard de paiment
SELECT 
    facture.reference AS facture,
    DATEDIFF(NOW(), facture.date_crea) AS nb_jours_retard
FROM facture
WHERE facture.date_paiement IS NULL
AND DATEDIFF(NOW(), facture.date_crea) > 30;
