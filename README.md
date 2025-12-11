#include <stdio.h>
#include <stdlib.h>
#include <time.h>

// Configuration (Tâche 1)
typedef struct {
    float seuil_min;
    float seuil_max;
    int intervalle;
} Config;

void lire_config(Config *cfg) {
    FILE *f = fopen("config.txt", "r");
    if (!f) {
        printf("config.txt introuvable → valeurs par défaut utilisées.\n");
        cfg->seuil_min = 18;
        cfg->seuil_max = 30;
        cfg->intervalle = 2;
        return;
    }
    fscanf(f, "%f %f %d", &cfg->seuil_min, &cfg->seuil_max, &cfg->intervalle);
    fclose(f);
}

//  Simulation capteur (Tâche 2) 
float lire_temperature() {
    return (rand() % 200) / 10.0 + 10;  // 10°C à 30°C
}

// Tâche 3 : Calcul du niveau d'alerte  

int niveau_alerte(float t, Config cfg) {
    if (t >= cfg.seuil_min && t <= cfg.seuil_max) return 0;

    float diff;
    if (t < cfg.seuil_min) diff = cfg.seuil_min - t;
    else diff = t - cfg.seuil_max;

    if (diff <= 1) return 1;
    if (diff <= 3) return 2;
    return 3;
}

//  Tâche 4 : Journalisation 
void log_event(float t, int niveau) {
    FILE *f = fopen("log.txt", "a");
    if (!f) return;

    time_t now = time(NULL);
    fprintf(f, "%s Temp=%.2f Niveau=%d\n", ctime(&now), t, niveau);
    fclose(f);
}

//  Fonction statistique
void afficher_statistiques(int count, float t, int niveau, int cons) {
    printf("Mesure %d : Temp=%.2f°C | Niveau=%d | Alertes consécutives=%d\n",
           count, t, niveau, cons);
}

//  Tâche 6 : Rapport journalier 
void generer_rapport(float min, float max, float moy, int n1, int n2, int n3) {
    FILE *f = fopen("rapport_journalier.txt", "w");
    if (!f) return;

    fprintf(f, "=== Rapport Journalier ===\n");
    fprintf(f, "Temp min : %.2f\n", min);
    fprintf(f, "Temp max : %.2f\n", max);
    fprintf(f, "Temp moyenne : %.2f\n", moy);
    fprintf(f, "Alertes Niveau 1 : %d\n", n1);
    fprintf(f, "Alertes Niveau 2 : %d\n", n2);
    fprintf(f, "Alertes Niveau 3 : %d\n", n3);
    fclose(f);
}

// --------- Programme principal ---------
int main() {
    srand(time(NULL));
    Config cfg;
    lire_config(&cfg);

    printf("Monitoring lancé... CTRL+C pour arrêter.\n");

    int cons = 0;               // compteur alertes consécutives
    int seuil_alertes = 3;      // nombre minimum d’alertes consécutives
    int n1 = 0, n2 = 0, n3 = 0; // compteur alertes par niveau
    float t, min = 999, max = -999, sum = 0;
    int count = 0;

    while (count < 20) {  // 20 mesures pour exemple
        t = lire_temperature();
        int niveau = niveau_alerte(t, cfg);

        //  Tâche 5 : Gestion des alertes consécutives 
        if (niveau > 0) cons++;
        else cons = 0;

        if (cons >= seuil_alertes) {
            switch(niveau) {
                case 1: n1++; printf("Avertissement léger ! Temp=%.2f\n", t); break;
                case 2: n2++; printf("Alerte modérée ! Temp=%.2f\n", t); break;
                case 3: n3++; printf("ALERTE CRITIQUE ! Temp=%.2f\n", t); break;
            }
        }

        //  Tâche 4 : Journalisation 
        log_event(t, niveau);

        // Statistiques pour rapport 
        if (t < min) min = t;
        if (t > max) max = t;
        sum += t;
        count++;

        // Fonction statistique
        afficher_statistiques(count, t, niveau, cons);
    }

    // --------- Tâche 6 : Génération du rapport (fonction séparée) ---------
    generer_rapport(min, max, sum / count, n1, n2, n3);

    printf("\nRapport généré dans rapport_journalier.txt\n");
    return 0;
}
