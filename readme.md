# Enhanced Educational Platform with RAG and Anxiety Detection

A comprehensive educational platform that combines multimodal Retrieval-Augmented Generation (RAG) with student personalization and anxiety detection features.

## Features

- **Multimodal RAG System**: Search and retrieve information from both text and images
- **Student Personalization**: Adapts content to student learning styles
- **Learning Analytics**: Tracks student engagement and provides personalized recommendations
- **Document Processing**: Upload and process PDF documents for text and image content
- **Anxiety Detection**: Assess and monitor anxiety levels through text and voice input
- **Customizable Collections**: Configure database collection names for different datasets

## System Requirements

- Python 3.8 or higher
- Docker (for Milvus)
- MongoDB 4.4 or higher
- Minimum 8GB RAM
- CUDA-compatible GPU (recommended for faster processing)

## Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/enhanced-educational-platform.git
cd enhanced-educational-platform
```

### 2. Set Up Python Environment

```bash
# Create a virtual environment
python -m venv venv

# Activate the virtual environment
# On Windows
venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate

# Install requirements
pip install -r requirements.txt
```

### 3. Set Up MongoDB

#### Install MongoDB

**On Ubuntu/Debian:**
```bash
# Import MongoDB public GPG key
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -

# Create list file for MongoDB
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

# Reload local package database
sudo apt-get update

# Install MongoDB packages
sudo apt-get install -y mongodb-org

# Start MongoDB service
sudo systemctl start mongod

# Enable MongoDB to start on boot
sudo systemctl enable mongod
```

**On macOS (using Homebrew):**
```bash
brew tap mongodb/brew
brew install mongodb-community@4.4
brew services start mongodb-community@4.4
```

**On Windows:**
1. Download the MongoDB installer from the [MongoDB Download Center](https://www.mongodb.com/try/download/community)
2. Run the installer and follow the installation wizard
3. Start MongoDB service from Services

#### Verify MongoDB Installation

```bash
# Connect to MongoDB shell
mongosh

# Check version
db.version()

# Exit shell
exit
```

### 4. Set Up Milvus with Docker

#### Install Docker

**On Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

**On macOS:**
Download and install Docker Desktop from [Docker's website](https://www.docker.com/products/docker-desktop)

**On Windows:**
Download and install Docker Desktop from [Docker's website](https://www.docker.com/products/docker-desktop)

#### Deploy Milvus using Docker Compose

1. Create a `docker-compose.yml` file:

```yaml
version: '3.5'

services:
  etcd:
    container_name: milvus-etcd
    image: quay.io/coreos/etcd:v3.5.0
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/etcd:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd

  minio:
    container_name: milvus-minio
    image: minio/minio:RELEASE.2020-12-03T00-03-10Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/minio:/minio_data
    command: minio server /minio_data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  standalone:
    container_name: milvus-standalone
    image: milvusdb/milvus:v2.0.0
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/milvus:/var/lib/milvus
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"

networks:
  default:
    name: milvus
```

2. Start Milvus:

```bash
docker-compose up -d
```

3. Verify Milvus is running:

```bash
docker ps | grep milvus
```

### 5. Application Configuration

1. Create a `.env` file in the project root:

```
# API Keys
GEMINI_API_KEY=your_gemini_api_key_here

# MongoDB Settings
MONGODB_URI=mongodb://localhost:27017/

# Milvus Settings
MILVUS_HOST=localhost
MILVUS_PORT=19530
```

2. Download the pre-trained anxiety detection model and place it in the project root:
   - Download `best_multimodal_model.pth` from the project's release assets or train your own model

### 6. Running the Application

```bash
# Activate the virtual environment if not already activated
# On Windows
venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate

# Run the Streamlit app
streamlit run main.py
```

The application will be available at http://localhost:8501

## Usage Guide

### Chat Interface

- Use the main chat interface to ask questions about your documents
- The system will retrieve relevant text and images from your documents
- View source information by enabling "Show text sources" in the sidebar
- Control image display with "Include images in responses" option

### Document Upload

1. **Text Processing**:
   - Fill in document metadata (Module Code, Module Name, Lecture Number, etc.)
   - Specify Text Collection Name or use the default
   - Upload a PDF document
   - Click "Process Document for Text"

2. **Image Processing**:
   - Configure database settings (MongoDB and Milvus collection names)
   - Fill in document metadata
   - Adjust advanced embedding settings if needed
   - Upload a PDF document
   - Click "Process PDF for Images"

### Learning Analytics

- Log in through the Student Personalization panel in the sidebar
- View analytics dashboards showing your learning patterns
- See recommendations based on your learning style and engagement
- Track progress across different modules

### Wellness Check

- Use the Wellness tab to assess anxiety levels
- Enter text or record voice for multimodal assessment
- Receive feedback on anxiety levels and helpful resources
- Track anxiety patterns over time (when logged in)

## Collection Customization

You can customize the database collections used by the application:

1. In the sidebar, under "Collection Settings":
   - Set "Text Collection" for text search
   - Set "MongoDB Images Collection" for image storage
   - Set "Milvus Images Collection" for image embeddings

2. When uploading documents:
   - Specify collection names to organize content by course, semester, etc.
   - Different collections can be used for different sets of documents

## Troubleshooting

### Common Issues

1. **MongoDB Connection Failed**:
   - Ensure MongoDB service is running: `sudo systemctl status mongod`
   - Check MongoDB URI in `.env` file

2. **Milvus Connection Failed**:
   - Verify Docker containers are running: `docker ps`
   - Check Milvus logs: `docker logs milvus-standalone`

3. **Model Loading Error**:
   - Ensure `best_multimodal_model.pth` is in the correct location
   - Check CUDA availability for GPU acceleration

4. **Audio Recording Issues**:
   - Install required system packages: `sudo apt-get install portaudio19-dev`
   - Verify microphone permissions in browser

### Getting Help

If you encounter issues not covered in this guide, please:
- Check the logs for error messages
- Refer to the project documentation
- File an issue on the project repository

## License

[Specify your license here]

## Acknowledgements

- This project uses [Gemini](https://ai.google.dev/gemini-api) for natural language processing
- Milvus for vector database functionality
- MongoDB for document storage
- Streamlit for the web interface
