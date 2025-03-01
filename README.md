# support-agent-chatbot
from flask import Flask, request, jsonify
from transformers import pipeline
import requests
from bs4 import BeautifulSoup

app = Flask(__name__)
qa_pipeline = pipeline("question-answering")

def fetch_documentation(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.text, "html.parser")
    return " ".join([p.text for p in soup.find_all("p")])

documentation = {
    "segment": fetch_documentation("https://segment.com/docs/"),
    "mparticle": fetch_documentation("https://docs.mparticle.com/"),
    "lytics": fetch_documentation("https://docs.lytics.com/"),
    "zeotap": fetch_documentation("https://docs.zeotap.com/home/en-us/")
}

@app.route("/ask", methods=["POST"])
def ask_question():
    data = request.json
    question = data.get("question")
    context = " ".join(documentation.values())
    answer = qa_pipeline(question=question, context=context)
    return jsonify(answer)

if __name__ == "__main__":
    app.run(debug=True)
