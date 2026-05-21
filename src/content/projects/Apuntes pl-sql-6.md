---
title: "Evidencia Clase 9 de Marzo"
publishDate: 2026-04-28
isFeatured: true
---

```sql

--- Triggers (Disparadores) Objetos que se ejecuran despues de una accion, utilizados para controlar un evento en la base de datos- se maneja en sentencias de tipo DML
--- CREATE OR REPLACE TRIGGER <nombre>
--- before|after : DELETE, INSERT, UPDATE
--- Siempre funciona un trigger para UNA SOLA tabla con ON
--- OF sirve para especificar columnas a analizar en la tabla
--- 2 varibles, ANTES DEL CAMBIO (OLD) Y DESPUES DEL CAMBIO (NEW)
--- INSERT CON OLD: NULO
--- INSERT CON NEW: NUEVO DATO
--- UPDATE CON OLD: Valores originales
--- UPDATE CON NEW: valores nuevo cuando se complete la orden
--- DELETE CON OLD: Valor borrado antes de 
--- DELTE CON NEW: NULO

--- Escalamiento horizontal: Cuando el server no tiene mas recursos (No se le puede aumentar memoria, etc) 
--- Escalamiento vertical: 16gb -> 32 gb

--- EXCEPCIONES
--- Sin declaracion y declaradas
--- PREFIJO: EXC_XXX


--- Excepciones en todo el trabajo.