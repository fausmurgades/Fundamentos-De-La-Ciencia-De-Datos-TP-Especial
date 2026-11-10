# TP Especial - Fundamentos de la Ciencia de Datos

Todo el código está en el notebook `entrega.ipynb`.

## Archivos

- `entrega.ipynb`: el notebook con todo el trabajo.
- `ENFR 2018 - Base usuario.txt`: la base de datos de la encuesta (las columnas están separadas por `|`).
- `diccionario_registros_enfr2018.xls` y `cuestionario_enfr_2018.pdf`: el diccionario de variables y el cuestionario de la encuesta. El código no los usa, nos sirvieron para entender qué significa cada variable.
- `requirements.txt`: las librerías que usamos, con sus versiones.

## Qué hace falta

- Python 3.14 (nosotros usamos la 3.14.9).
- Git para clonar el repo. 

## Pasos para correrlo

1. Clonar el repo y entrar a la carpeta:

   ```
   git clone https://github.com/fausmurgades/Fundamentos-De-La-Ciencia-De-Datos-TP-Especial.git
   cd Fundamentos-De-La-Ciencia-De-Datos-TP-Especial
   ```

2. Crear un entorno virtual (en Linux puede ser `python3` en vez de `python`):

   ```
   python -m venv venv
   ```

3. Activar el entorno:

   - Linux: `source venv/bin/activate`
   - Windows: `venv\Scripts\activate`

4. Instalar las librerías:

   ```
   pip install -r requirements.txt
   ```

5. Abrir el notebook (Jupyter ya se instala con el paso anterior):

   ```
   jupyter lab entrega.ipynb
   ```

   También se puede abrir con VS Code, eligiendo como kernel el Python de la carpeta `venv`.

6. Correr todas las celdas en orden, de arriba para abajo.

