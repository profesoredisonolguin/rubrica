<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rúbrica de Evaluación Personalizable</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f4f4f4;
        }
        .container {
            max-width: 1200px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            margin-bottom: 30px;
        }
        h1, h2 {
            text-align: center;
            color: #333;
        }
        #descriptor-selection-container {
            margin-bottom: 20px;
        }
        .course-selection-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        .course-selection-container label, .course-selection-container select {
            font-size: 16px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
            vertical-align: top;
            word-wrap: break-word;
        }
        th {
            background-color: #007bff;
            color: white;
            font-weight: bold;
        }
        .descriptor-table tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .descriptor-table th:first-child,
        .descriptor-table td:first-child {
            width: 40px;
            text-align: center;
        }
        .descriptor-table .max-score-selection {
            width: 80px;
            text-align: center;
        }
        .score-buttons, .score-buttons-selection {
            display: flex;
            gap: 5px;
            flex-wrap: wrap;
            justify-content: center;
            margin-top: 10px;
            width: 60px;
        }
        .score-buttons-selection .score-button {
            padding: 5px; /* Botones más pequeños para la selección */
        }
        .score-button {
            background-color: #eee;
            border: 1px solid #ccc;
            padding: 5px 10px;
            cursor: pointer;
            border-radius: 4px;
            transition: background-color 0.3s;
        }
        .score-button:hover {
            background-color: #ddd;
        }
        .score-button.selected {
            background-color: #28a745;
            color: white;
            border-color: #218838;
        }
        #generate-button {
            display: block;
            margin: 20px auto;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            transition: background-color 0.3s;
        }
        #generate-button:hover {
            background-color: #0056b3;
        }
        #rubrica-generada {
            margin-top: 30px;
            overflow-x: auto;
        }
        .compact-rubric-table {
            width: 100%;
            border-collapse: collapse;
        }
        .compact-rubric-table th,
        .compact-rubric-table td {
            text-align: center;
            vertical-align: middle;
            word-wrap: break-word;
            white-space: normal;
        }
        .compact-rubric-table th:first-child,
        .compact-rubric-table td:first-child {
            min-width: 120px;
            max-width: 200px;
            text-align: left;
        }
        .compact-rubric-table td input {
            width: 100%;
            box-sizing: border-box;
        }
        .compact-rubric-table .total-score-cell,
        .compact-rubric-table .final-grade-cell,
        .compact-rubric-table .total-score-header,
        .compact-rubric-table .final-grade-header {
            width: 65px;
            font-size: 1em;
            font-weight: bold;
            color: #333;
            background-color: #e9ecef;
            white-space: nowrap;
            padding: 8px;
        }
        .compact-rubric-table .total-score-header,
        .compact-rubric-table .final-grade-header {
            background-color: #007bff;
            color: white;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Rúbrica de Evaluación Personalizable</h1>

    <div id="descriptor-selection-container">
        <h2>Selecciona los indicadores con descriptores</h2>
        <p>Haz clic en el puntaje máximo para cada indicador que desees incluir en la rúbrica.</p>
        <table class="descriptor-table" id="descriptor-table">
            <thead>
                <tr>
                    <th class="max-score-selection">Máx</th>
                    <th>Indicadores</th>
                    <th>Nivel 4<br>(Excelente)</th>
                    <th>Nivel 3<br>(Bueno)</th>
                    <th>Nivel 2<br>(Regular)</th>
                    <th>Nivel 1<br>(Deficiente)</th>
                    <th>Nivel 0<br>(Ausencia Total)</th>
                </tr>
            </thead>
            <tbody>
            </tbody>
        </table>
    </div>

    <div id="rubrica-generacion-controls">
        <div class="course-selection-container">
            <div>
                <label for="course-select">Curso:</label>
                <select id="course-select"></select>
            </div>
            <div>
                <label for="letter-select">Letra:</label>
                <select id="letter-select"></select>
            </div>
        </div>
        <button id="generate-button">Generar Rúbrica</button>
    </div>

    <div id="rubrica-generada">
    </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', () => {
    const generateButton = document.getElementById('generate-button');
    const rubricaContainer = document.getElementById('rubrica-generada');
    const courseSelect = document.getElementById('course-select');
    const letterSelect = document.getElementById('letter-select');

    const ALL_INDICATORS_DATA = {
        'Postura corporal': {
            4: 'El estudiante mantiene una postura recta, que le permite una emisión de la voz adecuada.',
            3: 'Mantiene una postura cambiante, utilizando apoyos físicos de manera intermitente y manteniendo una postura que dificultan la emisión.',
            2: 'Mantiene una postura totalmente inadecuada. Se apoya en la pared, se sienta, se apoya en sus compañeros, etc.',
            1: '',
            0: ''
        },
        'Pulso': {
            4: 'El pulso es constante y preciso durante toda la canción.',
            3: 'El pulso es constante, la precisión no es la adecuada. No presenta dificultad para retomar el pulso si es que lo pierde.',
            2: 'El estudiante puede mantener un pulso, sin embargo lo pierde constantemente, logrando retomarlo con dificultad. La precisión no es adecuada.',
            1: 'No existe pulso constante durante la interpretación. No ha sido asimilado. No es capaz de retomar desde una determinada sección ni ingresar de manera precisa.',
            0: 'No ejecuta.'
        },
        'Disposición (actitudinal)': {
            4: 'El o la estudiante realiza el trabajo y manifiesta interés, escucha con atención, realiza preguntas, aporta comentarios, centra su mirada en el trabajo que se está realizando, busca y/o presta apoyo.',
            3: 'El o la estudiante realiza al menos 3 acciones de las descritas en el punto anterior.',
            2: 'El o la estudiante realiza al menos 2 acciones de las descritas anteriormente.',
            1: 'El o la estudiante se excusa o se rehúsa a participar, atrasando el proceso de evaluación y presenta solo 1 acción de las descritas anteriormente.',
            0: 'El o la estudiante se niega a realizar su evaluación.'
        },
        'Trabajo colaborativo': {
            4: 'El o la estudiante promueve la comunicación, aporta a organizar el grupo, escucha y acepta críticas, entrega opiniones asertivamente, apoya a sus compañeros, cumple con la función encomendada en el grupo.',
            3: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 4 acciones de las descritas en el punto anterior.',
            2: 'El o la estudiante realiza al menos entre 2 y 3 acciones de las descritas anteriormente.',
            1: 'El o la estudiante se excusa o se rehúsa a participar, atrasando el proceso de evaluación. Presenta solo 1 acción de las descritas anteriormente.',
            0: 'No participa ni realiza el trabajo.'
        },
        'Actitud al oír (actitudinal)': {
            4: 'El estudiante escucha atenta y respetuosamente la interpretación de sus compañeros.',
            3: 'El estudiante conversa mientras sus compañeros realizan su evaluación, sin embargo rectifica su actitud rápidamente.',
            2: 'El estudiante conversa mientras sus compañeros realizan la evaluación, su actitud genera desorden y distracción de sus compañeros.',
            1: 'El estudiante promueve el desorden interrumpiendo con ruidos en varias ocasiones, la evaluación de sus compañeros.',
            0: 'El estudiante se burla o interrumpe la evaluación de sus compañeros generando desorden o distracción de los evaluados.'
        },
        'Sigue las instrucciones (actitudinal)': {
            4: 'El estudiante presta atención a las indicaciones del profesor/a, las comprende (comprobándose con la actividad), hace consultas pertinentes y realiza la actividad de acuerdo a lo que se le solicita.',
            3: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 3 acciones de las descritas en el punto anterior.',
            2: 'El o la estudiante realiza al menos 2 acciones de las descritas anteriormente.',
            1: 'El o la estudiante no realiza la actividad de acuerdo a lo solicitado, presentando solo 1 acción de las descritas anteriormente.',
            0: 'El alumno no sigue las instrucciones.'
        },
        'Articulación': {
            4: 'El estudiante se esfuerza para que las frases musicales sean comprendidas. Aplica las técnicas aprendidas en las vocalizaciones.',
            3: '',
            2: 'El estudiante no aplica técnicas de articulación ni se preocupa de este elemento. La interpretación se entiende poco o nada.',
            1: '',
            0: 'No ejecuta.'
        },
        'Afinación': {
            4: 'El estudiante ejecuta de manera afinada.',
            3: 'El estudiante desafina en algunas ocasiones pero puede mantenerse en la tonalidad y retomar la interpretación.',
            2: 'El estudiante mantiene una afinación relativamente cercana, ejecuta los intervalos imprecisos.',
            1: 'El estudiante ejecuta de manera destemplada, no interpreta en la tonalidad y/o no ejecuta los intervalos sonoros.',
            0: 'No ejecuta.'
        },
        'Conocimiento letra': {
            4: 'El estudiante maneja totalmente la letra.',
            3: 'El estudiante presenta pocos errores, de los cuales se da cuenta y rectifica.',
            2: 'El estudiante conoce la letra pero se apoya demasiado en sus compañeros. Es un participante pasivo en la ejecución.',
            1: 'El estudiante presenta errores repetidos, lo que denota poco estudio, se apoya demasiado en sus compañeros. Ejecuta versos incompletos.',
            0: 'No ejecuta.'
        },
        'Emisión': {
            4: 'El estudiante emite un sonido adecuado, aplicando las técnicas de vocalización aprendidas.',
            3: 'El estudiante presenta algunos errores de emisión; engola, canta con muy poca intensidad lo que no permite su verificación, canta gritando, canta desde la garganta, etc.',
            2: '',
            1: '',
            0: 'No ejecuta.'
        },
        'Digitación': {
            4: 'El estudiante no presenta errores de digitación.',
            3: 'El estudiante no presenta errores de digitación sin embargo estas son débiles, lo que conlleva un sonido defectuoso.',
            2: 'El estudiante presenta 1 error de digitación que incluyen falencias en el sonido.',
            1: 'El estudiante presenta 2 o más errores de digitación que incluyen falencias en el sonido.',
            0: 'El estudiante desconoce totalmente la digitación del instrumento.'
        },
        'Lectura': {
            4: 'El estudiante no presenta errores de lectura.',
            3: 'El estudiante presenta errores, pero se percibe entendimiento en el proceso de lectura. (Identifica las notas corridas, identifica algunas notas, etc.)',
            2: 'El alumno no maneja el contenido.',
            1: '',
            0: ''
        },
        'Conocimiento melodía': {
            4: 'El estudiante ejecuta la melodía sin errores.',
            3: 'El estudiante comete errores que afectan mínimamente la estructura melódica.',
            2: 'El estudiante comete errores reiterativos lo que denota poco estudio, sin embargo se puede entender la melodía.',
            1: 'El estudiante comete errores en la melodía que termina desarmándola. No se encuentran motivos melódicos ni temas que permitan identificarla.',
            0: 'No ejecuta.'
        },
        'Originalidad': {
            4: 'El o la estudiante desarrolla una idea propia, aplica los conocimientos aprendidos, incluye elementos o técnicas propicias para la realización de la tarea, propone variados medios o formatos (visuales, corporales o auditivos) adecuados o pertinentes para expresar su idea.',
            3: '',
            2: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 3 acciones de las descritas en el punto anterior.',
            1: 'El o la estudiante realiza al menos 2 acciones de las descritas anteriormente.',
            0: 'El estudiante copia, sin embargo incluye elementos propios, realizando al menos 1 acción de las descritas anteriormente.'
        },
        'Creatividad': {
            4: 'El o a la estudiante propone diferentes ideas para realizar el trabajo, aplica criterios basados en los conocimientos aprendidos para discriminar los elementos que contendrá la tarea (decide que va a usar), organiza y planifica su proceso de creación, analiza y rectifica, si es necesario, su producto final.',
            3: '',
            2: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 3 acciones de las descritas en el punto anterior.',
            1: 'El o la estudiante realiza al menos 2 acciones de las descritas anteriormente.',
            0: 'El o la estudiante presenta solo 1 acción de las descritas anteriormente.'
        },
        'Representaciones gráficas': {
            4: 'El o la estudiante es capaz de establecer relaciones entre lo que escuchó y lo que dibuja, incluye en su dibujo varios sonidos, representa cualidades del sonido con formas o colores y relaciona sonoridades con emociones, sensaciones o experiencias.',
            3: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 3 acciones de las descritas en el punto anterior.',
            2: 'El o la estudiante realiza al menos 2 acciones de las descritas anteriormente.',
            1: 'El o la estudiante presenta solo 1 acción de las descritas anteriormente.',
            0: 'No realiza el trabajo.'
        },
        'Responsabilidad y compromiso (actitudinal)': {
            4: 'El o la estudiante cumple con las tareas solicitadas a tiempo, trae sus materiales, asiste a clases y/o justifica sus inasistencias, llega puntual, etc.',
            3: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 3 acciones de las descritas en el punto anterior.',
            2: 'El o la estudiante realiza las tareas solicitadas y cumple con al menos 2 acciones de las descritas anteriormente.',
            1: 'El o la estudiante no realiza las tareas solicitadas, pero cumple con al menos 1 acción de las descritas anteriormente.',
            0: 'El o la estudiante está presente en clases, pero no realiza las tareas solicitadas. No cumple con las acciones descritas anteriormente.'
        },
        'Acompañamiento rítmico': {
            4: 'El estudiante ejecuta el acompañamiento rítmico a pulso y sin errores.',
            3: 'El estudiante comete errores mínimamente perceptibles y es capaz de seguir con la interpretación.',
            2: 'El estudiante erra reiteradamente en la interpretación, lo que denota poco estudio, sin embargo a ratos puede incorporarse nuevamente al pulso.',
            1: 'El estudiante evidentemente no conoce el acompañamiento rítmico e improvisa su interpretación o deja el acompañamiento de lado.',
            0: 'No ejecuta.'
        },
        'Creación rítmica': {
            4: 'El estudiante crea patrones rítmicos respetando metro y simbología.',
            3: 'El estudiante comete errores mínimos de simbología y metro (menos de la mitad de los compases solicitados).',
            2: 'El estudiante no respeta metro ni simbología (en más de la mitad de la tarea solicitada).',
            1: 'El estudiante no respeta metro, ni simbología.',
            0: 'No realiza su trabajo.'
        }
    };

    // Estructura de datos de estudiantes (Parseada del XML que me proporcionaste, ordenada por nombre)
    // HE AÑADIDO MÁS DATOS DE EJEMPLO PARA OTROS CURSOS/LETRAS PARA PROBAR LA FUNCIONALIDAD
    // DEBES REEMPLAZAR ESTO CON TU DATA COMPLETA Y CORRECTA EXTRAÍDA DEL XML
    const ALL_STUDENTS_DATA = {
        "1°A": ["AGUILERA ESPINOZA, JOAQUÍN BENJAMÍN", "AGUILERA ESPINOZA, RENATO ANDRÉS", "ALARCÓN BASCUR, IGNACIO CRISTÓBAL"],
        "1°B": ["ESTUDIANTE 1 B", "ESTUDIANTE 2 B", "ESTUDIANTE 3 B"],
        "2°A": ["ALMONACID CÁCERES, ÁNGEL DAVID", "ALVARADO ORTIZ, MAXIMILIANO CRISTÓBAL", "ARANEDA ALVEAL, EMILIO ALONSO"],
        "2°B": ["RAMÍREZ, ANA", "SOTO, PEDRO", "VEGA, LUIS"],
        "3°A": ["ACUÑA VALENZUELA, BENJAMÍN ALONSO", "AGUILERA ROJAS, MATÍAS IGNACIO", "ALARCÓN MUÑOZ, ANTONIA BELÉN"],
        "3°B": ["CONTRERAS, SOFÍA", "DÍAZ, MARIO", "ESPINOZA, LAURA", "GONZALEZ, JOAQUIN", "HERRERA, VALENTINA"],
        "4°A": ["ABARCA ROJAS, JOSEFA ANTONIA", "ACUÑA CONTRERAS, VICENTE IGNACIO", "AGUILERA ALARCÓN, MATÍAS ANDRÉS"],
        "4°B": ["GARCIA, ANDRES", "LOPEZ, DANIELA", "MARTINEZ, PABLO"],
        "5°A": ["ABARCA ROJAS, JOSEFA ANTONIA", "ACUÑA CONTRERAS, VICENTE IGNACIO", "AGUILERA ALARCÓN, MATÍAS ANDRÉS"],
        "5°B": ["MOLINA, FELIPE", "ORTEGA, CAMILA", "PEREZ, SEBASTIÁN"],
        "6°A": ["ABARCA ROJAS, JOSEFA ANTONIA", "ACUÑA CONTRERAS, VICENTE IGNACIO", "AGUILERA ALARCÓN, MATÍAS ANDRÉS"],
        "6°B": ["RODRIGUEZ, ANTONIA", "SANCHEZ, IGNACIO", "TAPIA, EMILIA"],
        "7°A": ["ABARCA ROJAS, JOSEFA ANTONIA", "ACUÑA CONTRERAS, VICENTE IGNACIO", "AGUILERA ALARCÓN, MATÍAS ANDRÉS"],
        "7°B": ["VARGAS, DIEGO", "ZAPATA, FRANCISCA", "CASTRO, BENJAMIN"],
        "8°A": ["ABARCA ROJAS, JOSEFA ANTONIA", "ACUÑA CONTRERAS, VICENTE IGNACIO", "AGUILERA ALARCÓN, MATÍAS ANDRÉS"],
        "8°B": ["DIAZ, NICOLAS", "FLORES, SOFIA", "GOMEZ, JAVIERA"]
    };

    // Estructura de cursos y letras disponibles para los selectores (generada dinámicamente)
    const ALL_COURSES_AND_LETTERS = {};
    for (const key in ALL_STUDENTS_DATA) {
        // Extraer el número del curso (ej. "1°") y la letra (ej. "A")
        // Esta regex es más flexible para capturar el número y el símbolo de grado
        const match = key.match(/^(\d+°)([A-Z])$/);
        if (match) {
            const courseNum = match[1]; // ej. "1°"
            const letter = match[2];    // ej. "A"
            if (!ALL_COURSES_AND_LETTERS[courseNum]) {
                ALL_COURSES_AND_LETTERS[courseNum] = [];
            }
            if (!ALL_COURSES_AND_LETTERS[courseNum].includes(letter)) {
                ALL_COURSES_AND_LETTERS[courseNum].push(letter);
            }
        }
    }
    // Asegurarse de que las letras dentro de cada curso estén ordenadas alfabéticamente
    for (const courseNum in ALL_COURSES_AND_LETTERS) {
        ALL_COURSES_AND_LETTERS[courseNum].sort();
    }


    function populateCourseAndLetterSelectors() {
        courseSelect.innerHTML = ''; // Clear existing options

        // Ordenar los cursos numéricamente (ej. "1°", "2°", "10°")
        const sortedCourses = Object.keys(ALL_COURSES_AND_LETTERS).sort((a, b) => {
            const numA = parseInt(a.replace('°', ''));
            const numB = parseInt(b.replace('°', ''));
            return numA - numB;
        });

        sortedCourses.forEach(course => {
            const option = document.createElement('option');
            option.value = course;
            option.textContent = course;
            courseSelect.appendChild(option);
        });

        // Populate letter selector based on initial course
        populateLetterSelector();

        // Add event listener to update letter selector when course changes
        courseSelect.addEventListener('change', populateLetterSelector);
    }

    function populateLetterSelector() {
        letterSelect.innerHTML = ''; // Clear existing options
        const selectedCourse = courseSelect.value;
        const letters = ALL_COURSES_AND_LETTERS[selectedCourse];
        
        if (letters && letters.length > 0) {
            letterSelect.disabled = false;
            letters.forEach(letter => {
                const option = document.createElement('option');
                option.value = letter;
                option.textContent = letter;
                letterSelect.appendChild(option);
            });
        } else {
            const option = document.createElement('option');
            option.value = '';
            option.textContent = 'N/A';
            letterSelect.appendChild(option);
            letterSelect.disabled = true; // Disable if no letters are available
        }
    }

    function renderDescriptorTable() {
        const tableBody = document.querySelector('#descriptor-table tbody');
        tableBody.innerHTML = '';

        for (const indicator in ALL_INDICATORS_DATA) {
            const data = { ...ALL_INDICATORS_DATA[indicator] };
            
            // Lógica para reubicar descriptores según tus instrucciones
            if (indicator === 'Articulación') {
                const temp4 = data[4];
                const temp2 = data[2];
                data[4] = '';
                data[3] = '';
                data[2] = temp4;
                data[1] = temp2;
            } else if (indicator === 'Emisión') {
                const temp4 = data[4];
                const temp3 = data[3];
                data[4] = '';
                data[3] = '';
                data[2] = temp4;
                data[1] = temp3;
            } else if (indicator === 'Lectura') {
                const temp4 = data[4];
                const temp3 = data[3];
                const temp2 = data[2];
                data[4] = '';
                data[3] = '';
                data[2] = temp4;
                data[1] = temp3;
                data[0] = temp2;
            } else if (indicator === 'Originalidad' || indicator === 'Creatividad') {
                const temp2 = data[2];
                const temp1 = data[1];
                const temp0 = data[0];
                data[3] = temp2;
                data[2] = temp1;
                data[1] = temp0;
                data[0] = 'No ejecuta.';
            } else if (indicator === 'Postura corporal') {
                const temp4 = data[4];
                const temp3 = data[3];
                const temp2 = data[2];
                data[4] = '';
                data[3] = '';
                data[2] = temp4;
                data[1] = temp3;
                data[0] = temp2;
            }

            const row = document.createElement('tr');
            row.dataset.indicator = indicator;
            row.innerHTML = `
                <td class="max-score-selection">
                    <div class="score-buttons-selection">
                        <button class="score-button" data-score="4">4</button>
                        <button class="score-button" data-score="2">2</button>
                    </div>
                </td>
                <td><strong>${indicator}</strong></td>
                <td>${data[4] || ''}</td>
                <td>${data[3] || ''}</td>
                <td>${data[2] || ''}</td>
                <td>${data[1] || ''}</td>
                <td>${data[0] || ''}</td>
            `;
            tableBody.appendChild(row);
        }
        addSelectionEventListeners();
    }

    function addSelectionEventListeners() {
        document.querySelectorAll('.descriptor-table .score-buttons-selection .score-button').forEach(button => {
            button.addEventListener('click', (event) => {
                const row = event.target.closest('tr');
                const scoreButtonsInRow = row.querySelectorAll('.score-buttons-selection .score-button');
                scoreButtonsInRow.forEach(btn => btn.classList.remove('selected'));
                event.target.classList.add('selected');
            });
        });
    }

    // Inicializar selectores y tabla de descriptores
    renderDescriptorTable();
    populateCourseAndLetterSelectors();

    generateButton.addEventListener('click', () => {
        const selectedIndicatorsAndScores = Array.from(document.querySelectorAll('.descriptor-table tr')).reduce((acc, row) => {
            const selectedButton = row.querySelector('.score-button.selected');
            if (selectedButton) {
                acc.push({
                    indicator: row.dataset.indicator,
                    maxScore: parseInt(selectedButton.dataset.score, 10)
                });
            }
            return acc;
        }, []);

        if (selectedIndicatorsAndScores.length === 0) {
            alert('Por favor, selecciona al menos un indicador.');
            return;
        }

        const selectedCourseValue = courseSelect.value; 
        const selectedLetterValue = letterSelect.value;
        const fullCourseKey = `${selectedCourseValue}${selectedLetterValue}`; // Ej: "3°A"

        const students = ALL_STUDENTS_DATA[fullCourseKey];

        let headerHTML = `<th>Nombre</th>`;
        selectedIndicatorsAndScores.forEach(item => {
            headerHTML += `<th>${item.indicator}</th>`;
        });
        headerHTML += `<th class="total-score-header">Puntaje Total</th>`;
        headerHTML += `<th class="final-grade-header">Nota</th>`;

        let bodyHTML = '';
        if (students && students.length > 0) {
            students.forEach(studentName => {
                bodyHTML += `<tr>`;
                bodyHTML += `<td>${studentName}</td>`;
                selectedIndicatorsAndScores.forEach(item => {
                    bodyHTML += `<td data-indicador="${item.indicator}" data-max-score="${item.maxScore}"><div class="score-buttons"></div></td>`;
                });
                bodyHTML += `<td class="total-score-cell">0</td>`;
                bodyHTML += `<td class="final-grade-cell">2.0</td>`;
                bodyHTML += `</tr>`;
            });
        } else {
            bodyHTML = `<tr><td colspan="${selectedIndicatorsAndScores.length + 3}">No se encontraron alumnos para el curso ${fullCourseKey}. Asegúrate de que los datos estén correctamente cargados.</td></tr>`;
        }

        const tableHTML = `
            <h2>Rúbrica Generada para ${fullCourseKey}</h2>
            <table class="compact-rubric-table">
                <thead>
                    <tr>${headerHTML}</tr>
                </thead>
                <tbody>${bodyHTML}</tbody>
            </table>
        `;

        rubricaContainer.innerHTML = tableHTML;

        // Si hay estudiantes, inicializar la lógica de puntuación
        if (students && students.length > 0) {
            initScoreLogic(selectedIndicatorsAndScores);
        }
    });

    function initScoreLogic(indicatorsAndScores) {
        const rows = rubricaContainer.querySelectorAll('.compact-rubric-table tbody tr');
        // Filtra las filas que no son de estudiante (ej. "No se encontraron alumnos...")
        const studentRows = Array.from(rows).filter(row => row.querySelector('td[data-indicador]'));

        const totalMaxScore = indicatorsAndScores.reduce((sum, item) => sum + item.maxScore, 0);
        // Evitar división por cero si no hay indicadores seleccionados
        const scoreFor40 = totalMaxScore > 0 ? totalMaxScore * 0.5 : 0; 

        // Función para calcular la nota final
        function calculateGrade(currentTotalScore) {
            let finalGrade;
            if (totalMaxScore === 0) return 'N/A'; // No hay indicadores, no hay nota posible

            if (currentTotalScore <= scoreFor40) {
                finalGrade = 2.0 + (currentTotalScore / scoreFor40) * 2.0;
            } else {
                finalGrade = 4.0 + ((currentTotalScore - scoreFor40) / (totalMaxScore - scoreFor40)) * 3.0;
            }
            finalGrade = Math.max(2.0, Math.min(7.0, finalGrade));
            return finalGrade.toFixed(1);
        }

        studentRows.forEach(row => {
            const scoreCells = row.querySelectorAll('td[data-indicador]');
            const totalScoreElement = row.querySelector('.total-score-cell');
            const finalGradeElement = row.querySelector('.final-grade-cell');
            let scores = {};

            scoreCells.forEach((cell) => {
                const indicator = cell.dataset.indicador;
                const maxScore = parseInt(cell.dataset.maxScore, 10);
                scores[indicator] = 0; // Inicializar puntaje en 0 para cada indicador
                const scoreButtonsContainer = cell.querySelector('.score-buttons');

                if (scoreButtonsContainer) {
                    scoreButtonsContainer.innerHTML = '';
                    for (let i = maxScore; i >= 0; i--) {
                        const button = document.createElement('button');
                        button.className = 'score-button';
                        button.textContent = i;
                        button.dataset.score = i;
                        button.addEventListener('click', () => {
                            handleScoreSelection(cell, button, i);
                        });
                        scoreButtonsContainer.appendChild(button);
                    }
                }
            });

            function handleScoreSelection(cell, selectedButton, score) {
                const indicator = cell.dataset.indicador;
                cell.querySelectorAll('.score-button').forEach(button => {
                    button.classList.remove('selected');
                });
                selectedButton.classList.add('selected');
                scores[indicator] = score;
                updateTotalScore();
            }

            function updateTotalScore() {
                const total = Object.values(scores).reduce((sum, current) => sum + current, 0);
                totalScoreElement.textContent = total;
                finalGradeElement.textContent = calculateGrade(total);
            }

            // Asegurarse de que el total inicial y la nota se muestren correctamente
            updateTotalScore();
        });
    }
});
</script>

</body>
</html>
