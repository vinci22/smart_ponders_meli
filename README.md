# smart_ponders_meli


# README — Sistema de Ponderación de Métricas (MELI)

Este documento resume el modelo de ponderación propuesto, con  **fórmulas en notación matemática** , definición de  **variables** , y la  **conexión con el EDA** . Está pensado como guía para implementación y defensa técnica del enfoque.

---

## 1) Objetivo

Construir un **índice ponderado por usuario** que refleje la **relevancia relativa** de su actividad, corrigiendo sesgos de **volumen** y  **redundancia** , e incorporando  **informatividad** , **engagement** y **estructura latente** (PCA).

---

## 2) Notación y variables

* Conjuntos y contadores:
  * C\mathcal{C}: conjunto de *clusters* (grupos funcionales).
  * Mc\mathcal{M}_c: conjunto de métricas dentro del *cluster* cc.
  * U\mathcal{U}: conjunto de usuarios.
  * Nusuarios=∣U∣N_{usuarios} = |\mathcal{U}|.
* Agregados por *cluster* cc:
  * EventoscEventos_c: suma de eventos en el cluster cc, ∑m∈Mc∑u∈Uevent_count[u,m]\sum_{m\in \mathcal{M}_c}\sum_{u\in \mathcal{U}} event\_count[u,m].
  * Diversidadc=∣Mc∣Diversidad_c = |\mathcal{M}_c|: número de métricas únicas en cc.
* Agregados por métrica mm en *cluster* cc:
  * Eventosm,c=∑u∈Uevent_count[u,m]Eventos_{m,c} = \sum_{u\in \mathcal{U}} event\_count[u,m].
  * Usuarios_activosm=∣{u∈U:event_count[u,m]>0}∣Usuarios\_activos_m = |\{u\in \mathcal{U}: event\_count[u,m] > 0\}|.
  * ρ(m,m′)\rho(m, m'): correlación (por ejemplo, Pearson) entre métricas mm y m′m' dentro del mismo cluster.
* Factores de ajuste:
  * share_high[m]share\_high[m]: proporción de eventos de mm generados por usuarios de alto  *engagement* .
  * share_all[m]share\_all[m]: proporción de eventos de mm sobre el total de la población.
  * PCA: loadingm,kloading_{m,k} (carga de mm en componente kk), var_expkvar\_exp_k (varianza explicada del componente kk), KK (número de componentes retenidos), γ\gamma (hiperparámetro).

---

## 3) Ponderación de *clusters*

**Fórmula:**

wc=Eventosc⋅log⁡(1+Diversidadc)∑c′∈CEventosc′⋅log⁡(1+Diversidadc′)w_c = \frac{Eventos_c \cdot \log\big(1 + Diversidad_c\big)}{\sum\limits_{c'\in\mathcal{C}} Eventos_{c'} \cdot \log\big(1 + Diversidad_{c'}\big)}
**Intuición:** integra **volumen** (EventoscEventos_c) y **riqueza** (DiversidadcDiversidad_c), amortiguando el efecto de diversidad con log⁡(⋅)\log(\cdot).

---

## 4) Ponderación de métricas dentro de un *cluster*

### 4.1 Frecuencia relativa (importancia local)

fm ∣ c=Eventosm,c∑m′∈McEventosm′,c f_{m\,|\,c} = \frac{Eventos_{m,c}}{\sum\limits_{m'\in \mathcal{M}_c} Eventos_{m',c}}

### 4.2 Informatividad (tipo IDF)

idfm=log⁡(1+Nusuarios1+Usuarios_activosm) idf_m = \log\Bigg(1 + \frac{N_{usuarios}}{1 + Usuarios\_activos_m}\Bigg)

### 4.3 Penalización por redundancia (correlación)

penm=11+promedio⁡( ∣ρ(m,m′)∣ )  para m′∈Mc,m′≠m pen_m = \frac{1}{1 + \operatorname{promedio}\big(\,|\rho(m, m')|\,\big)\ \ \text{para } m'\in\mathcal{M}_c, m'\neq m}

### 4.4 Peso normalizado en el *cluster*

w~m ∣ c=fm ∣ c⋅(1+idfm)⋅penm,wm ∣ c=w~m ∣ c∑m′∈Mcw~m′ ∣ c \tilde{w}_{m\,|\,c} = f_{m\,|\,c} \cdot \big(1 + idf_m\big) \cdot pen_m, \qquad
 w_{m\,|\,c} = \frac{\tilde{w}_{m\,|\,c}}{\sum\limits_{m'\in \mathcal{M}_c} \tilde{w}_{m'\,|\,c}}
**Intuición:** volumen local (ff) + capacidad de discriminar (idfidf) + unicidad (penpen).

---

## 5) Ajustes avanzados (globales por métrica)

### 5.1 Ajuste por *engagement*

feng[m]=share_high[m]share_all[m] f_{eng}[m] = \frac{share\_high[m]}{share\_all[m]}

> Si los usuarios de alto *engagement* usan relativamente más mm, el factor >1>1; si no, <1<1.

### 5.2 Ajuste por PCA (estructura latente)

fpca[m]=1+γ⋅z ⁣( ∑k≤Kloadingm,k 2 ⋅ var_expk) f_{pca}[m] = 1 + \gamma \cdot z\!\left(\,\sum\limits_{k\le K} loading_{m,k}^{\,2}\,\cdot\, var\_exp_k\right)

* z(⋅)z(\cdot): estandarización (zz-score) para hacer comparables los valores entre métricas.
* γ\gamma: controla cuánto pesa el ajuste PCA (usualmente 0.20.2–1.01.0).

---

## 6) *Score* final por usuario

### 6.1 Peso total de la métrica

wmtotal=wc⋅wm ∣ c⋅feng[m]⋅fpca[m] w^{\text{total}}_m = w_c \cdot w_{m\,|\,c} \cdot f_{eng}[m] \cdot f_{pca}[m]

### 6.2 Índice ponderado

Scoreu=∑c∈C ∑m∈Mc(wmtotal⋅event_count[u,m]) Score_u = \sum\limits_{c\in\mathcal{C}}\ \sum\limits_{m\in\mathcal{M}_c} \bigg( w^{\text{total}}_m \cdot event\_count[u,m] \bigg)

### 6.3 Baseline plano y normalización

Scoreuplano=∑c∈C ∑m∈Mcevent_count[u,m] Score^{\text{plano}}_u = \sum\limits_{c\in\mathcal{C}}\ \sum\limits_{m\in\mathcal{M}_c} event\_count[u,m]
Scoreunorm=100⋅Scoreu−min⁡u′Scoreu′max⁡u′Scoreu′−min⁡u′Scoreu′ Score^{\text{norm}}_u = 100\cdot \frac{Score_u - \min\limits_{u'} Score_{u'}}{\max\limits_{u'} Score_{u'} - \min\limits_{u'} Score_{u'}}
Δu=Scoreunorm−(Scoreuplano)norm \Delta_u = Score^{\text{norm}}_u - (Score^{\text{plano}}_u)^{\text{norm}}
**Lectura:** Δu\Delta_u muestra cuánto cambia la posición del usuario al pasar de un conteo plano a uno ponderado.

---

## 7) Conexión explícita con el EDA

* **Sesgo por *cluster*** → wcw_c (volumen + diversidad via log⁡\log).
* **Diferencias de valor dentro del *cluster*** → wm ∣ cw_{m\,|\,c} (frecuencia + informatividad + no redundancia).
* **Preferencias de top users** → feng[m]f_{eng}[m].
* **Estructura latente y factores globales** → fpca[m]f_{pca}[m].

---

## 8) Parámetros y *tuning*

* γ\gamma (PCA): [0.2,1.0][0.2, 1.0]. Ajustar con validación (por ejemplo, estabilidad del *ranking* y capacidad de discriminar cohorts conocidas).
* Criterio de correlación para penmpen_m: Pearson por defecto; explorar Spearman si hay no linealidades.
* Definición de cohort  *high engagement* : percentil superior (p. ej., 80–90) del ScoreplanoScore^{\text{plano}} o reglas de negocio.

---

## 9) Supuestos y consideraciones

* Las métricas están correctamente asignadas a un único  *cluster* .
* Las correlaciones se calculan con series agregadas de métricas comparables (mismo *scaling* o transformaciones consistentes).
* idfmidf_m asume que la **rareza** aporta poder de discriminación; si todas las métricas están ampliamente usadas, su efecto será pequeño.

---

## 10) Flujo de implementación (resumen)

1. Calcular EventoscEventos_c, DiversidadcDiversidad_c → wcw_c.
2. Para cada cc: calcular fm ∣ cf_{m\,|\,c}, idfmidf_m, penmpen_m → wm ∣ cw_{m\,|\,c}.
3. Calcular feng[m]f_{eng}[m] (definir cohort) y fpca[m]f_{pca}[m] (retener KK para ≥80% varianza, *tunar* γ\gamma).
4. Obtener wmtotalw^{\text{total}}_m y luego ScoreuScore_u para cada usuario.
5. Normalizar, comparar con baseline y analizar Δu\Delta_u.

---

## 11) Glosario rápido

* **Frecuencia relativa** : peso local de una métrica dentro de su  *cluster* .
* **IDF (informatividad)** : recompensa rareza/selección por subconjuntos de usuarios.
* **Penalización por redundancia** : reduce el peso de métricas duplicadas informativamente.
* **Engagement** : preferencia de uso por la cohorte de alto desempeño.
* **PCA** : refuerza métricas que explican la varianza global en factores latentes.
