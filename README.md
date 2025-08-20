# 📊 Challenge: Modelo de Ponderación de Métricas de Actividad Digital

## 1. Contexto

Este proyecto aborda el desafío de **Mercado Libre**: construir un modelo que refleje de manera más representativa el **comportamiento digital de los usuarios** dentro de un ecosistema de herramientas.  

Cada usuario genera eventos asociados a métricas específicas (ej. *commits, dashboards creados, consultas ejecutadas*), las cuales pertenecen a un **clúster funcional** (*software development, data analysis, documentation*, etc.).  

El punto de partida es una lógica plana: todas las métricas y clusters pesan lo mismo. El objetivo es **superar esa simplificación**, diseñando un **sistema de ponderación avanzado** que capture mejor la relevancia relativa de cada acción.

---

## 2. Objetivo del Proyecto

- **Asignar ponderaciones diferenciadas** a:  
  1. **Clusters funcionales** (según volumen y diversidad).  
  2. **Métricas dentro de cada cluster** (según frecuencia, entropía, correlación, y capacidad de discriminación por rol/seniority).  

- **Construir un índice ponderado por usuario** que:  
  - Corrija sesgos de volumen y redundancia.  
  - Destaque acciones significativas sobre interacciones triviales.  
  - Sea adaptable a distintos perfiles de usuarios.  

---

## 3. Enfoque Metodológico

### a) Exploración de Datos (EDA)
- Revisión de calidad y consistencia (cardinalidades, valores nulos, duplicados).  
- Análisis de distribución de eventos por **cluster** y por **métrica dentro de cluster**.  
- Identificación de **skew** (clusters dominantes), métricas poco usadas (ruido), y redundancia (correlación alta entre métricas).  
- Aplicación de **PCA** para detectar estructura latente en la actividad de usuarios.  

### b) Sistema de Ponderación

**1. Ponderación de clusters**  

$$
w_c = \frac{Eventos_c \cdot \log(1 + Diversidad_c)}{\sum_{c'} Eventos_{c'} \cdot \log(1 + Diversidad_{c'})}
$$  

- $Eventos_c$: volumen total del cluster.  
- $Diversidad_c$: cantidad de métricas distintas activas.  
- Ajusta clusters grandes y premia los diversos.  

---

**2. Ponderación de métricas dentro del cluster**  

$$
w_{m|c} \propto f_{m|c} \cdot H_m \cdot D_{m}
$$  

- $f_{m|c}$: frecuencia relativa de la métrica en el cluster.  
- $H_m$: entropía → mide dispersión entre usuarios.  
- $D_m$: discriminación → qué tan bien separa perfiles de rol/seniority.  

---

**3. Correcciones adicionales**  
- Penalización a métricas redundantes (correlación alta).  
- Bonus a métricas con baja correlación (aportan información única).  
- Ajustes por **rol/seniority** para evitar sesgos de perfil.  
- Integración de **PCA** para reforzar métricas que explican varianza latente.  

---

**4. Score por usuario**  

$$
Score_u = \sum_{c \in C} w_c \cdot \sum_{m \in M_c} w_{m|c} \cdot event\_count_{u,m}
$$

---

## 4. Resultados

- El modelo ponderado revela **diferencias sustanciales** respecto al baseline plano.  
- Usuarios con **mucho volumen pero poca diversidad** bajan en el ranking.  
- Usuarios con **eventos más balanceados y significativos** suben.  
- La introducción de entropía y discriminación permite capturar la **calidad de la actividad**, no solo la cantidad.

---

## 5. Aprendizajes

- La ponderación necesita apoyarse en **datos exploratorios sólidos**: volumen, diversidad y correlación fueron claves.  
- **No todas las métricas son comparables**: algunas capturan valor estratégico (ej. *commits, dashboards*) y otras son triviales (*likes, comentarios*).  
- El uso de **PCA y entropía** permitió separar señales de ruido y robustecer la ponderación.  

---

## 6. Mejoras futuras

- Integrar señales de **impacto real en negocio** (ej. despliegues en producción, adopción de dashboards).  
- Incorporar **modelos supervisados** con etiquetas de “alto desempeño” para ajustar ponderaciones.  
- Explorar temporalidad: métricas recientes vs históricas.  

---

## 7. Estructura del Repo

