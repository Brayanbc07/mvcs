# mvcs
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Dashboard de Proyectos de Agua Potable y Alcantarillado</title>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
</head>
<body>
    <h1>Dashboard de Proyectos de Agua Potable y Alcantarillado</h1>
    <label for="selector">Selecciona una métrica:</label>
    <select id="selector"></select>
    <div id="grafica" style="width:100%; max-width:1000px; height:600px;"></div>

    <script>
        // Ruta al archivo CSV (debes subirlo a GitHub o servidor y ajustar el path)
        const csvUrl = "Presupuesto_PNSU.csv";

        // Lista de métricas a mostrar si están en los datos
        const opciones = ["AVNCE (%)", "DEVENGADO", "PIM", "COSTO DEL PROYECTO"];

        fetch(csvUrl)
            .then(response => response.text())
            .then(data => {
                const parsed = Papa.parse(data, { header: true });
                const df = parsed.data;

                // Validar que se haya leído correctamente
                if (!df.length) {
                    document.body.innerHTML += "<p style='color:red'>No se pudo cargar el archivo CSV.</p>";
                    return;
                }

                // Obtener las columnas disponibles
                const columnas = Object.keys(df[0]);
                const opcionesValidas = opciones.filter(op => columnas.includes(op));
                const selector = document.getElementById("selector");

                if (!opcionesValidas.length) {
                    document.body.innerHTML += "<p style='color:red'>Ninguna de las métricas esperadas está en el archivo CSV.</p>";
                    return;
                }

                // Rellenar el select con las métricas válidas
                opcionesValidas.forEach(opcion => {
                    const opt = document.createElement("option");
                    opt.value = opcion;
                    opt.textContent = opcion;
                    selector.appendChild(opt);
                });

                // Función para renderizar el gráfico
                function renderChart(metric) {
                    const proyectos = [];
                    const valores = [];

                    df.forEach(row => {
                        const nombre = row["PROYECTO"];
                        const valor = parseFloat(row[metric]);

                        if (nombre && !isNaN(valor)) {
                            proyectos.push(nombre);
                            valores.push(valor);
                        }
                    });

                    // Orden descendente por defecto
                    const sorted = proyectos.map((p, i) => [valores[i], p])
                        .sort((a, b) => a[0] - b[0]);

                    const trace = {
                        x: sorted.map(d => d[0]),
                        y: sorted.map(d => d[1]),
                        type: "bar",
                        orientation: "h"
                    };

                    const layout = {
                        title: `${metric} por proyecto`,
                        margin: { l: 200 }
                    };

                    Plotly.newPlot("grafica", [trace], layout);
                }

                // Escuchar cambios en el selector
                selector.addEventListener("change", () => {
                    renderChart(selector.value);
                });

                // Graficar por defecto con la primera métrica
                renderChart(opcionesValidas[0]);
            })
            .catch(error => {
                document.body.innerHTML += `<p style='color:red'>Error al cargar los datos: ${error}</p>`;
            });
    </script>
</body>
</html>
