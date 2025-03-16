# Start-Hack-M-nstack
Model preparation and deploy  for the hackthon

# Domain Adaptation Model API

A lightweight API service for domain-adapted language models that preserves natural language capabilities while adding domain-specific knowledge.

## 🌟 Features

- **Domain Adaptation**: Fine-tune pre-trained language models on specific domains while preserving general language abilities
- **Simple API**: Easy-to-use FastAPI service with clear endpoints
- **Production-Ready**: Detailed deployment guides for VPS with Nginx/Apache
- **Modular Design**: Easily extensible codebase for hackathons and projects

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- PyTorch 1.9+
- CUDA (optional, for GPU acceleration)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/domain-adaptation-api.git
cd domain-adaptation-api

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Running the API Server (Development)

```bash
# Start the API server
uvicorn api.simple_api:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at http://localhost:8000. You can access the automatic documentation at http://localhost:8000/docs.

### Training a Domain-Adapted Model

```bash
# Run the training script with your domain data
python scripts/train_model.py --domain medical --data data/medical_texts.csv --epochs 5
```

## 📚 API Usage

### Load a Model

```bash
curl -X POST "http://localhost:8000/load-model" \
  -H "Content-Type: application/json" \
  -d '{"model_path": "models/medical_20250316_123456"}'
```

### Predict Masked Tokens

```bash
curl -X POST "http://localhost:8000/predict-mask" \
  -H "Content-Type: application/json" \
  -d '{"text": "The patient'"'"'s [MASK] showed signs of myocardial infarction.", "k": 5}'
```

## 📋 Project Structure

```
domain-adaptation-api/
├── api/                   # API code
│   ├── simple_api.py      # Simple API implementation
│   ├── routers/           # API routers (advanced version)
│   └── ...
├── models/                # Directory for saved models
├── scripts/               # Training and utility scripts
│   ├── train_model.py     # Domain adaptation training script
│   └── ...
├── deploy/                # Deployment configurations
│   ├── nginx/             # Nginx configuration
│   ├── apache/            # Apache configuration
│   └── systemd/           # Systemd service files
├── docs/                  # Documentation
│   ├── deployment.md      # Deployment guide
│   └── api.md             # API reference
└── requirements.txt       # Project dependencies
```

## 🔧 Deployment

See [Deployment Guide](docs/deployment.md) for detailed instructions on deploying this API service to production using:

- VPS (DigitalOcean, AWS EC2, etc.)
- Nginx as a reverse proxy
- Apache as an alternative to Nginx
- Systemd for process management
- SSL/TLS configuration

## 📊 Performance Considerations

- For optimal performance, use a machine with a GPU.
- Model loading takes significant memory - adjust batch sizes according to your hardware.
- Consider using a smaller model for faster inference on limited hardware.

## 🔗 Related Projects

- [PEFT](https://github.com/huggingface/peft) - Parameter-Efficient Fine-Tuning methods
- [Hugging Face Transformers](https://github.com/huggingface/transformers) - State-of-the-art Natural Language Processing

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgements

- Hugging Face for the Transformers library
- FastAPI for the web framework
