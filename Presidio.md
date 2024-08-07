
import os
from flask import Flask, request, jsonify
from presidio_analyzer import AnalyzerEngine
from presidio_anonymizer import AnonymizerEngine
from transformers import pipeline

# Initialize Flask app
app = Flask(__name__)

# Initialize AnalyzerEngine
analyzer = AnalyzerEngine()

# Initialize AnonymizerEngine
anonymizer = AnonymizerEngine()

# Example Transformers pipeline for Named Entity Recognition (NER)
ner_pipeline = pipeline('ner')

@app.route('/analyze', methods=['POST'])
def analyze():
    data = request.json
    text = data.get("text")
    language = data.get("language", "en")

    if not text:
        return jsonify({"error": "Text is required"}), 400

    # Use the analyzer engine to analyze the text
    analyzer_results = analyzer.analyze(text=text, language=language)
    return jsonify(analyzer_results)

@app.route('/anonymize', methods=['POST'])
def anonymize():
    data = request.json
    text = data.get("text")
    analyzer_results = data.get("analyzer_results")

    if not text or not analyzer_results:
        return jsonify({"error": "Text and analyzer_results are required"}), 400

    # Use the anonymizer engine to anonymize the text
    anonymizer_results = anonymizer.anonymize(text=text, analyzer_results=analyzer_results)
    return jsonify(anonymizer_results)

@app.route('/ner', methods=['POST'])
def ner():
    data = request.json
    text = data.get("text")

    if not text:
        return jsonify({"error": "Text is required"}), 400

    # Use the Transformers NER pipeline to analyze the text
    ner_results = ner_pipeline(text)
    return jsonify(ner_results)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
