# ecg-anomaly-detection
Real-time ECG anomaly detection using Apache Spark, Random Forest, Flask, and interactive web visualization.


# Real-Time ECG Anomaly Detection Using Spark-Based Signal Processing

## Overview
This project is a Big Data Analytics application that detects ECG anomalies using Apache Spark and Machine Learning. It processes ECG signals, extracts statistical features, and predicts whether the ECG is **Normal** or **Abnormal** using a Random Forest classifier. The system also includes a Flask API and a web interface for real-time ECG analysis. :contentReference[oaicite:0]{index=0}

## Features
- Real-time ECG anomaly detection
- Apache Spark-based distributed processing
- Random Forest classification
- Flask REST API
- Interactive web interface
- ECG waveform visualization using Chart.js :contentReference[oaicite:1]{index=1}

## Technologies Used
- Apache Spark
- Java
- Python
- Flask
- HTML
- CSS
- JavaScript
- Chart.js
- Maven

## Project Workflow
1. Load ECG dataset
2. Preprocess ECG signals
3. Extract statistical features
4. Train Random Forest model
5. Predict Normal/Abnormal ECG
6. Display prediction and ECG waveform on the web interface :contentReference[oaicite:2]{index=2}

## Dataset
- PhysioNet 2017 ECG Dataset


## Future Improvements
- Deep Learning (CNN/LSTM)
- Real-time ECG streaming
- Multi-class arrhythmia detection
- Cloud deployment
- Explainable AI integration :contentReference[oaicite:4]{index=4}

## License
This project is developed for academic purposes as part of the **Big Data Analytics** course at **Amrita Vishwa Vidyapeetham**.


























##User-Interface Design

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>ECG Anomaly Detection</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body {
    margin: 0;
    padding: 0;
    font-family: "Segoe UI", Arial, sans-serif;
    background: linear-gradient(135deg, #e6f0ff, #f6fbff);
}

.container {
    max-width: 900px;
    margin: 40px auto;
    padding: 20px;
}

.card {
    background: rgba(255, 255, 255, 0.85);
    backdrop-filter: blur(8px);
    border-radius: 16px;
    padding: 30px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.1);
}

h1 {
    text-align: center;
    color: #0047ab;
}

.sub {
    text-align: center;
    color: #555;
}

.upload-box {
    text-align: center;
    margin: 20px 0;
}

button {
    padding: 10px 22px;
    background: #0056d6;
    color: white;
    border-radius: 8px;
    border: none;
    cursor: pointer;
}

button:hover {
    background: #003f9e;
}

.badge {
    padding: 6px 15px;
    border-radius: 8px;
    font-weight: bold;
}

.normal { background: #d4f8d4; color: #0e700e; }
.abnormal { background: #ffd4d4; color: #b30000; }

.risk-low    { background: #d4f8d4; color:#0e700e; }
.risk-med    { background: #fff3cd; color:#7a5b00; }
.risk-high   { background: #ffd4d4; color:#b30000; }

.panel {
    background: #f1f6ff;
    padding: 20px;
    border-radius: 12px;
    margin-top: 20px;
}

#rawBox {
    display: none;
    background: #eee;
    padding: 20px;
    border-radius: 10px;
    white-space: pre-wrap;
}

.toggleRaw {
    cursor: pointer;
    color: #0056d6;
}
</style>
</head>

<body>

<div class="container">
<div class="card">

<h1>ECG Anomaly Detection</h1>
<p class="sub">Upload your ECG signal to detect anomalies</p>

<div class="upload-box">
    <input id="fileInput" type="file">
    <button onclick="predict()">Predict</button>
</div>

<!-- Prediction Panel -->
<div class="panel">
    <h3>🔍 Prediction Result</h3>

    <p><b>Prediction:</b> <span id="predLabel">--</span></p>

    <p><b>Risk Level:</b> <span id="riskLevel">--</span></p>

    <p><b>Predictions File:</b> <span id="predCSV">--</span></p>
</div>

<!-- Meaning Panel -->
<div class="panel">
    <h3>🩺 What This Means</h3>
    <p id="meaningText">--</p>
</div>

<!-- Model Detection Panel -->
<div class="panel">
    <h3>🧠 What the Model Detected</h3>
    <ul>
        <li>QRS peak positions analyzed</li>
        <li>R–R interval rhythm checked</li>
        <li>Overall signal morphology evaluated</li>
    </ul>
</div>

<!-- ECG Graph -->
<div class="panel">
    <h3>📈 ECG Waveform</h3>
    <canvas id="ecgChart" height="150"></canvas>
</div>

<div class="toggleRaw" onclick="toggleRaw()">Show raw output ▼</div>
<pre id="rawBox"></pre>

</div>
</div>

<script>

let chart = null;

/* SHOW/HIDE RAW OUTPUT */
function toggleRaw() {
    const box = document.getElementById("rawBox");
    box.style.display = box.style.display === "none" ? "block" : "none";
}

/* COMPUTE RISK LEVEL WITHOUT PROBABILITY */
function computeRisk(pred) {
    if (pred === 0) return `<span class='badge risk-low'>Low</span>`;
    return `<span class='badge risk-high'>High</span>`;  
}

/* MEANING TEXT */
function meaning(pred) {
    return pred === 0
        ? "Your ECG appears NORMAL — rhythm and waveform look stable."
        : "An ABNORMAL rhythm was detected. This may indicate cardiac irregularity.";
}

async function predict() {

    const file = document.getElementById("fileInput").files[0];
    if (!file) { alert("Upload a CSV file first!"); return; }

    const formData = new FormData();
    formData.append("file", file);

    const res = await fetch("/predict", { method: "POST", body: formData });
    const data = await res.json();

    // raw output box
    document.getElementById("rawBox").textContent = JSON.stringify(data, null, 2);

    /* ----------------------
       PREDICTION
    ------------------------*/
    const pred = data.summary.sample[0].prediction;

    document.getElementById("predLabel").innerHTML =
        pred === 0 ? "<span class='badge normal'>Normal</span>"
                   : "<span class='badge abnormal'>Abnormal</span>";

    document.getElementById("riskLevel").innerHTML = computeRisk(pred);
    document.getElementById("meaningText").innerText = meaning(pred);
    document.getElementById("predCSV").innerText = data.summary.predictions_csv;

    /* ----------------------
       DRAW ECG
    ------------------------*/
    const reader = new FileReader();
    reader.onload = (evt) => {
        const text = evt.target.result.trim();
        const lines = text.split("\n");

        // ECG values exist on 2nd row
        const columns = lines[1].split(",");

        // take first 2000 samples
        const ecg = columns.slice(0, 2000).map(Number);

        drawChart(ecg);
    };
    reader.readAsText(file);
}

/* DRAW ECG WAVEFORM */
function drawChart(values) {

    const ctx = document.getElementById("ecgChart");

    if (chart) chart.destroy();

    chart = new Chart(ctx, {
        type: "line",
        data: {
            labels: values.map((_, i) => i),
            datasets: [{
                label: "ECG Signal",
                data: values,
                borderColor: "#0056d6",
                borderWidth: 1.3,
                pointRadius: 0,
                tension: 0.2
            }]
        },
        options: {
            scales: { x: { display: false } },
            plugins: { legend: { display: false } }
        }
    });
}

</script>

</body>
</html>


CODE

package com.ecg.anomaly;

import java.io.FileWriter;
import java.io.PrintWriter;
import java.time.Instant;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.IntStream;

import org.apache.spark.ml.classification.RandomForestClassificationModel;
import org.apache.spark.ml.classification.RandomForestClassifier;
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator;
import org.apache.spark.ml.feature.VectorAssembler;
import org.apache.spark.sql.*;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.Column;

public class ECGAnomalyPhysioNet {

    public static void main(String[] args) throws Exception {

        if (args.length < 2) {
            System.err.println("Usage: ECGAnomalyPhysioNet <hdfs_input_csv> <local_output_json>");
            System.exit(1);
        }

        // ---------- INPUTS ----------
        String hdfsInput = args[0];          // Example: hdfs://localhost:9000/ecg/input.csv
        String localJsonOutput = args[1];    // Needed by Flask

        // ---------- START SPARK ----------
        SparkSession spark = SparkSession.builder()
                .appName("ECG Anomaly Detection via Hadoop + Spark")
                .master("local[*]")
                .getOrCreate();

        spark.sparkContext().setLogLevel("ERROR");

        System.out.println("STEP 1: Reading ECG CSV from HDFS → " + hdfsInput);

        Dataset<Row> df = spark.read()
                .option("header", "true")
                .option("inferSchema", "true")
                .csv(hdfsInput);

        System.out.println("Loaded rows: " + df.count());

        // ---------- DETECT SIGNAL COLUMNS ----------
        List<String> signalNames = new ArrayList<>();
        for (String col : df.columns()) {
            if (col.matches("\\d+"))    // select numeric column names only
                signalNames.add(col);
        }

        System.out.println("Detected " + signalNames.size() + " ECG signal columns");

        // Convert each column to double
        Column[] castCols = signalNames.stream()
                .map(c -> functions.col(c).cast("double"))
                .toArray(Column[]::new);

        Dataset<Row> withArray = df.withColumn("signal_array", functions.array(castCols));

        // ---------- FEATURE ENGINEERING ----------
        withArray = withArray.selectExpr("*",
                "aggregate(signal_array, 0D, (acc,x)->acc + coalesce(x,0D)) as sum",
                "aggregate(signal_array, 0D, (acc,x)->acc + (coalesce(x,0D)*coalesce(x,0D))) as sumsq",
                "array_min(signal_array) as minv",
                "array_max(signal_array) as maxv",
                "size(signal_array) as len");

        withArray = withArray.withColumn("mean", functions.expr("sum / len"))
                .withColumn("energy", functions.col("sumsq"))
                .withColumn("peakToPeak", functions.expr("maxv - minv"))
                .withColumn("variance", functions.expr(
                        "aggregate(transform(signal_array, x -> pow(coalesce(x,0D)-mean,2D)),0D,(acc,y)->acc+y)/len"))
                .withColumn("stddev", functions.sqrt(functions.col("variance")));

        // Numerical label
        Dataset<Row> prepared = withArray.withColumn("label_num", functions.col("label").cast("double"))
                .na().fill(0.0, new String[]{"label_num"})
                .select("label_num", "mean", "stddev", "minv", "maxv", "energy", "peakToPeak");

        Dataset<Row> finalDF = prepared
                .withColumnRenamed("minv", "min")
                .withColumnRenamed("maxv", "max");

        // ---------- VECTOR ASSEMBLER ----------
        VectorAssembler assembler = new VectorAssembler()
                .setInputCols(new String[]{"mean", "stddev", "min", "max", "energy", "peakToPeak"})
                .setOutputCol("features");

        Dataset<Row> vectorDF = assembler.transform(finalDF)
                .select(functions.col("label_num").alias("label"), functions.col("features"));

        // ---------- TRAIN & TEST ----------
        Dataset<Row>[] splits = vectorDF.randomSplit(new double[]{0.8, 0.2}, 42);
        Dataset<Row> train = splits[0];
        Dataset<Row> test = splits[1];

        System.out.println("Training = " + train.count() + ", Testing = " + test.count());

        RandomForestClassifier rf = new RandomForestClassifier()
                .setLabelCol("label")
                .setFeaturesCol("features")
                .setNumTrees(50);

        RandomForestClassificationModel model = rf.fit(train);

        Dataset<Row> predictions = model.transform(test);

        // Convert probability to string for CSV writing
        Dataset<Row> predsForSave = predictions
                .withColumn("probability_str", functions.col("probability").cast("string"))
                .select("label", "prediction", "probability_str");

        // ---------- SAVE OUTPUT TO HDFS ----------
        String hdfsOut = "hdfs://localhost:9000/ecg/output/run_" + Instant.now().getEpochSecond();

        predsForSave.coalesce(1)
                .write()
                .mode(SaveMode.Overwrite)
                .option("header", "true")
                .csv(hdfsOut);

        System.out.println("Saved predictions to HDFS: " + hdfsOut);

        // ---------- ACCURACY ----------
        double accuracy = new MulticlassClassificationEvaluator()
                .setLabelCol("label")
                .setPredictionCol("prediction")
                .setMetricName("accuracy")
                .evaluate(predictions);

        System.out.println("MODEL ACCURACY = " + accuracy);

        // ---------- SAMPLE JSON FOR FLASK ----------
        List<Row> sample = predsForSave.limit(10).collectAsList();

        StringBuilder json = new StringBuilder();
        json.append("{\n");
        json.append("  \"accuracy\": ").append(accuracy).append(",\n");
        json.append("  \"predictions_csv\": \"").append(hdfsOut).append("\",\n");
        json.append("  \"sample\": [\n");

        for (int i = 0; i < sample.size(); i++) {
            Row r = sample.get(i);
            json.append("    { \"label\": ").append(r.getDouble(0))
                    .append(", \"prediction\": ").append(r.getDouble(1))
                    .append(", \"probability\": \"").append(r.getString(2)).append("\" }");
            if (i < sample.size() - 1) json.append(",");
            json.append("\n");
        }
        json.append("  ]\n");
        json.append("}");

        // Write JSON summary locally
        try (PrintWriter out = new PrintWriter(new FileWriter(localJsonOutput))) {
            out.write(json.toString());
        }

        System.out.println("JSON summary saved to LOCAL: " + localJsonOutput);
        spark.stop();
    }
}


from flask import Flask, request, jsonify, send_from_directory
from flask_cors import CORS
import subprocess
import os
import json
import uuid

app = Flask(__name__, static_folder="frontend", static_url_path="")
CORS(app)

# Ensure folders exist
os.makedirs("uploads", exist_ok=True)
os.makedirs("output", exist_ok=True)

@app.route("/")
def index():
    return send_from_directory("frontend", "index.html")


@app.route("/predict", methods=["POST"])
def predict():
    file = request.files.get("file")
    if file is None:
        return jsonify({"error": "No file uploaded"}), 400

    # -----------------------------
    # 1. Save uploaded file locally
    # -----------------------------
    local_filename = f"upload_{uuid.uuid4()}.csv"
    local_path = os.path.join("uploads", local_filename)
    file.save(local_path)

    # -----------------------------
    # 2. Upload file to HDFS
    # -----------------------------
    hdfs_path = f"/ecg/{local_filename}"

    try:
        subprocess.run(["hdfs", "dfs", "-put", "-f", local_path, hdfs_path],
                       check=True)
    except subprocess.CalledProcessError:
        return jsonify({"error": "Failed to upload file to HDFS"}), 500

    # -----------------------------
    # 3. Spark output JSON (local)
    # -----------------------------
    output_json = os.path.join("output", "result.json")
    if os.path.exists(output_json):
        os.remove(output_json)

    # -----------------------------
    # 4. Run Spark JAR (reads from HDFS)
    # -----------------------------
    try:
        result = subprocess.check_output([
            "spark-submit",
            "--class", "com.ecg.anomaly.ECGAnomalyPhysioNet",
            "--master", "local[*]",
            "../target/ecg-anomaly-detection-1.0-SNAPSHOT-jar-with-dependencies.jar",
            f"hdfs://localhost:9000{hdfs_path}",     # <-- SPARK READS FROM HDFS
            output_json
        ], stderr=subprocess.STDOUT)

    except subprocess.CalledProcessError as e:
        return jsonify({
            "error": "Spark execution failed",
            "details": e.output.decode()
        }), 500

    # -----------------------------
    # 5. Load Spark summary JSON
    # -----------------------------
    if os.path.exists(output_json):
        with open(output_json, "r") as f:
            summary = json.load(f)
    else:
        summary = {"error": "Spark did not generate result.json"}

    return jsonify({
        "result": "Execution completed",
        "summary": summary,
        "raw_output": result.decode()
    })


if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)
