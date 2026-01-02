# Chapter 4: System Design

## 4.1 System Perspective / Architectural Design

The Voice Cloning and Speech Synthesis System is designed as a modular, end-to-end neural network-based architecture that transforms textual input into natural-sounding speech using cloned voice characteristics. The system follows a layered architecture pattern, separating concerns between data processing, neural model inference, and user interaction layers.

### High-Level Architecture

The system architecture consists of five primary layers:

1. **Presentation Layer**: Gradio-based web interface for user interaction
2. **Application Layer**: Request handling, preprocessing orchestration, and response formatting
3. **Processing Layer**: Audio feature extraction, text tokenization, and post-processing
4. **Model Layer**: Neural network components (Tacotron2 encoder-decoder, vocoder)
5. **Data Layer**: Voice embeddings storage, model checkpoints, and audio file management

```mermaid
graph TB
    subgraph "Presentation Layer"
        UI[Gradio Web Interface]
        Upload[File Upload Component]
        Player[Audio Player]
    end
    
    subgraph "Application Layer"
        API[API Controller]
        ReqHandler[Request Handler]
        RespFormatter[Response Formatter]
    end
    
    subgraph "Processing Layer"
        AudioProc[Audio Preprocessor]
        TextProc[Text Processor]
        PostProc[Post Processor]
        Silence[Silence Trimmer]
    end
    
    subgraph "Model Layer"
        Encoder[Text Encoder<br/>LSTM]
        Attention[Attention Mechanism]
        Decoder[Mel-Spec Decoder]
        Vocoder[WaveGlow Vocoder]
        SpkEnc[Speaker Encoder]
    end
    
    subgraph "Data Layer"
        VoiceDB[(Voice Embeddings)]
        Models[(Model Checkpoints)]
        AudioFS[Audio File Storage]
    end
    
    UI --> Upload
    UI --> Player
    Upload --> API
    API --> ReqHandler
    ReqHandler --> AudioProc
    ReqHandler --> TextProc
    AudioProc --> SpkEnc
    TextProc --> Encoder
    Encoder --> Attention
    Attention --> Decoder
    SpkEnc --> Decoder
    Decoder --> Vocoder
    Vocoder --> PostProc
    PostProc --> Silence
    Silence --> RespFormatter
    RespFormatter --> Player
    
    SpkEnc -.-> VoiceDB
    Encoder -.-> Models
    Decoder -.-> Models
    Vocoder -.-> Models
    PostProc --> AudioFS
    
    style UI fill:#e1f5ff
    style API fill:#fff4e1
    style AudioProc fill:#f0e1ff
    style Encoder fill:#ffe1e1
    style VoiceDB fill:#e1ffe1
```

### Architecture Components

**Presentation Layer Components:**
- **Web Interface**: Gradio-based UI providing file upload, parameter controls, and audio playback
- **Upload Handler**: Manages reference audio file uploads (WAV, MP3, FLAC formats)
- **Visualization**: Real-time generation logs and audio waveform display

**Application Layer Components:**
- **API Controller**: Routes requests between interface and processing layers
- **Request Handler**: Validates inputs, manages session state, and coordinates processing pipeline
- **Response Formatter**: Structures generated audio, timestamps, and metadata for client delivery

**Processing Layer Components:**
- **Audio Preprocessor**: Resamples audio to 24kHz, converts to mono, normalizes amplitude
- **Feature Extractor**: Computes mel-spectrograms (80 bins) and MFCCs from audio waveforms
- **Text Processor**: Tokenizes input text, handles character-level encoding
- **Post Processor**: Applies silence trimming, audio normalization, and format conversion

**Model Layer Components:**
- **Text Encoder**: Bidirectional LSTM network (256 hidden units) encoding text sequences
- **Attention Mechanism**: Location-sensitive attention aligning text and audio representations
- **Decoder**: Autoregressive decoder generating mel-spectrograms with Prenet/Postnet layers
- **Vocoder**: WaveGlow neural vocoder converting spectrograms to audio waveforms
- **Speaker Encoder**: ResNet-based network extracting voice embeddings (192-dimensional vectors)

**Data Layer Components:**
- **Voice Embeddings Database**: Stores extracted speaker embeddings for voice cloning
- **Model Checkpoints**: Persists trained model weights and configuration
- **Audio Storage**: File system for reference audio and generated outputs

### Deployment Architecture

The system supports deployment in cloud-based notebook environments (Kaggle, Google Colab) with GPU acceleration:

```mermaid
graph LR
    subgraph "Development Environment"
        Colab[Google Colab/<br/>Kaggle Notebook]
        GPU[T4/P100 GPU]
        Storage[Persistent Storage]
    end
    
    subgraph "Runtime Components"
        Python[Python 3.11<br/>Runtime]
        PyTorch[PyTorch 2.1<br/>CUDA 12.1]
        Gradio[Gradio<br/>Interface]
    end
    
    subgraph "External Access"
        Public[Public URL<br/>via Gradio Share]
        Users[End Users]
    end
    
    Colab --> Python
    Python --> PyTorch
    PyTorch --> GPU
    Python --> Gradio
    Gradio --> Public
    Public --> Users
    Python -.-> Storage
    
    style Colab fill:#f9f9f9
    style GPU fill:#ffe1e1
    style Gradio fill:#e1f5ff
    style Public fill:#e1ffe1
```

## 4.2 Context Diagram

The context diagram illustrates the system's boundaries and interactions with external entities. The Voice Cloning System operates as a self-contained unit processing user inputs and generating audio outputs.

```mermaid
graph TB
    User[👤 End User<br/>Content Creator/Producer]
    Admin[👤 System Administrator<br/>ML Engineer]
    
    subgraph System["Voice Cloning & Speech Synthesis System"]
        Core[Core Processing Engine]
    end
    
    RefAudio[(Reference Audio<br/>Files)]
    Models[(Pre-trained<br/>Model Weights)]
    Output[(Generated<br/>Audio Files)]
    Config[(Configuration<br/>Parameters)]
    
    User -->|1. Upload reference audio| Core
    User -->|2. Input text script| Core
    User -->|3. Set parameters| Core
    Core -->|4. Generated speech| User
    Core -->|5. Timestamps & metadata| User
    
    Admin -->|Configure model| Core
    Admin -->|Monitor performance| Core
    Core -->|System logs| Admin
    
    Core -->|Read| RefAudio
    Core -->|Load| Models
    Core -->|Write| Output
    Core -->|Read| Config
    
    RefAudio -.->|Voice samples| User
    Output -.->|Downloads| User
    
    style User fill:#e1f5ff
    style Admin fill:#fff4e1
    style System fill:#f0e1ff
    style Core fill:#ffe1e1
```

### Context Description

**External Entities:**

1. **End User (Content Creator/Producer)**
   - Uploads reference audio clips (10-30 seconds) of target voice
   - Inputs text scripts for speech synthesis
   - Configures generation parameters (CFG scale, silence trimming)
   - Receives generated audio files and associated metadata

2. **System Administrator (ML Engineer)**
   - Manages model configurations and hyperparameters
   - Monitors system performance and resource utilization
   - Reviews generation logs and error reports
   - Performs model updates and maintenance

**External Data Stores:**

1. **Reference Audio Files**: User-provided voice samples in various formats (WAV, MP3, FLAC)
2. **Pre-trained Model Weights**: Neural network checkpoints and configuration files
3. **Generated Audio Files**: System output including synthesized speech and timestamps
4. **Configuration Parameters**: System settings, hyperparameters, and user preferences

**System Boundaries:**

The system boundary encompasses all neural network processing, audio manipulation, and interface components. External to the system are the users, raw input files, and persistent storage. The system does not include video processing, real-time streaming services, or external API integrations.

**Data Flow Summary:**

- **Input Flow**: Reference audio → Feature extraction → Voice embedding
- **Processing Flow**: Text + Voice embedding → Neural synthesis → Audio waveform
- **Output Flow**: Generated audio → Post-processing → File storage → User download

---

# Chapter 5: Detailed Design

## 5.1 System Design

The detailed system design elaborates on the internal architecture, focusing on component interactions, data transformations, and processing workflows. This section provides implementation-level specifications for each major subsystem.

### Neural Network Architecture

The core of the system is built around a sequence-to-sequence architecture with attention mechanisms, specifically designed for text-to-speech synthesis with voice cloning capabilities.

```mermaid
graph TD
    subgraph "Input Processing"
        TextIn[Text Input<br/>'Hello World']
        AudioIn[Reference Audio<br/>24kHz WAV]
        TextEnc[Character Tokenizer<br/>Vocab: 100 chars]
        AudioFeat[Feature Extractor<br/>Mel-Spec: 80 bins]
    end
    
    subgraph "Encoder Network"
        CharEmbed[Character Embedding<br/>Dimension: 512]
        ConvLayers[3x Conv1D Layers<br/>Kernel: 5, Filters: 512]
        BiLSTM[Bi-LSTM Layers<br/>2 layers, 256 units each]
        EncOut[Encoder Output<br/>Shape: [T, 512]]
    end
    
    subgraph "Speaker Embedding"
        SpkNet[ResNet Speaker Encoder<br/>18 layers]
        SpkEmbed[Speaker Embedding<br/>192-dimensional vector]
        SpkTile[Tile & Concatenate<br/>to each decoder step]
    end
    
    subgraph "Attention & Decoder"
        Attention[Location-Sensitive<br/>Attention]
        Prenet[Prenet<br/>2x FC: 256 units]
        DecLSTM[Decoder LSTM<br/>2 layers, 1024 units]
        Postnet[Postnet<br/>5x Conv1D: 512 filters]
        MelOut[Mel-Spectrogram<br/>Shape: [N, 80]]
    end
    
    subgraph "Vocoder"
        WaveGlow[WaveGlow Vocoder<br/>12 coupling layers]
        AudioOut[Audio Waveform<br/>24kHz, Float32]
    end
    
    TextIn --> TextEnc
    TextEnc --> CharEmbed
    CharEmbed --> ConvLayers
    ConvLayers --> BiLSTM
    BiLSTM --> EncOut
    
    AudioIn --> AudioFeat
    AudioFeat --> SpkNet
    SpkNet --> SpkEmbed
    
    EncOut --> Attention
    SpkEmbed --> SpkTile
    Attention --> Prenet
    SpkTile --> DecLSTM
    Prenet --> DecLSTM
    DecLSTM --> Postnet
    Postnet --> MelOut
    
    MelOut --> WaveGlow
    WaveGlow --> AudioOut
    
    style TextIn fill:#e1f5ff
    style AudioIn fill:#e1f5ff
    style EncOut fill:#fff4e1
    style SpkEmbed fill:#f0e1ff
    style MelOut fill:#ffe1e1
    style AudioOut fill:#e1ffe1
```

### Component Specifications

**Text Encoder:**
- **Input**: Character sequence (max length: 1000)
- **Character Embedding**: 512-dimensional vectors
- **Convolutional Layers**: 3 layers, kernel size 5, 512 filters, ReLU activation
- **Bi-LSTM**: 2 layers, 256 hidden units per direction (512 total)
- **Output**: Encoded sequence of shape [sequence_length, 512]
- **Purpose**: Captures linguistic context and phonetic information from text

**Speaker Encoder:**
- **Architecture**: ResNet-18 backbone modified for audio input
- **Input**: Mel-spectrogram of reference audio (80 mel bins × time frames)
- **Layers**: 18 convolutional layers with residual connections
- **Output**: 192-dimensional speaker embedding vector
- **Purpose**: Extracts voice-specific characteristics (timbre, pitch, speaking style)
- **Training**: Pre-trained on speaker verification task for robust embeddings

**Attention Mechanism:**
- **Type**: Location-sensitive attention with cumulative weights
- **Query**: Current decoder hidden state (1024-dim)
- **Keys/Values**: Encoder outputs (512-dim)
- **Location Features**: 1D convolution on attention weights (32 filters, kernel 31)
- **Alignment**: Soft attention weights over input sequence
- **Purpose**: Aligns decoder output with corresponding input text positions

**Decoder Network:**
- **Prenet**: 2 fully-connected layers (256 units each), dropout 0.5, ReLU
- **LSTM**: 2 layers, 1024 units per layer, unidirectional
- **Input**: Previous mel-frame + speaker embedding + attention context
- **Output**: Predicted mel-frame (80 bins)
- **Postnet**: 5 Conv1D layers (512 filters), batch norm, tanh, residual connection
- **Purpose**: Autoregressively generates mel-spectrogram frames

**WaveGlow Vocoder:**
- **Architecture**: Flow-based generative model with 12 coupling layers
- **Coupling**: Affine transformation with WaveNet-style dilated convolutions
- **Input**: Mel-spectrogram (80 bins × frames)
- **Output**: Raw audio waveform (24kHz sample rate)
- **Parameters**: ~88M parameters trained for high-fidelity synthesis
- **Inference**: Parallel generation (non-autoregressive) for speed

### Processing Pipeline Design

```mermaid
sequenceDiagram
    participant User
    participant UI as Gradio Interface
    participant API as API Controller
    participant Proc as Audio Processor
    participant Model as Neural Model
    participant Storage as File Storage
    
    User->>UI: Upload reference audio
    UI->>API: POST /upload {audio_file}
    API->>Proc: Validate & preprocess audio
    Proc->>Proc: Resample to 24kHz, normalize
    Proc->>Model: Extract speaker embedding
    Model-->>Proc: 192-dim embedding vector
    Proc->>Storage: Save voice profile
    Storage-->>UI: Voice added to dropdown
    
    User->>UI: Enter text & select voice
    User->>UI: Click "Generate Audio"
    UI->>API: POST /generate {text, voice_id, params}
    API->>Proc: Tokenize text
    Proc->>Model: Load voice embedding
    Model->>Model: Encode text sequence
    Model->>Model: Apply attention & decode
    Model->>Model: Generate mel-spectrogram
    Model->>Model: Vocoder: mel → waveform
    Model-->>Proc: Audio waveform [N samples]
    Proc->>Proc: Trim silence (optional)
    Proc->>Proc: Normalize audio
    Proc->>Storage: Save output.wav
    Storage-->>UI: Return audio file + metadata
    UI-->>User: Display audio player & download
    
    Note over Model: Processing time: 1-5 minutes<br/>depending on text length
```

### Data Flow Architecture

```mermaid
flowchart LR
    subgraph Input["Input Stage"]
        A1[Text Script]
        A2[Reference Audio]
    end
    
    subgraph Preprocessing["Preprocessing Stage"]
        B1[Character Tokenization]
        B2[Audio Resampling<br/>24kHz Mono]
        B3[Mel-Spectrogram<br/>Extraction]
    end
    
    subgraph Encoding["Encoding Stage"]
        C1[Text Embedding<br/>512-dim]
        C2[Convolutional<br/>Encoding]
        C3[Bi-LSTM<br/>Encoding]
        C4[Speaker<br/>Embedding]
    end
    
    subgraph Synthesis["Synthesis Stage"]
        D1[Attention<br/>Alignment]
        D2[Decoder<br/>LSTM]
        D3[Mel-Spectrogram<br/>Generation]
        D4[WaveGlow<br/>Vocoder]
    end
    
    subgraph Output["Output Stage"]
        E1[Audio Waveform]
        E2[Silence Trimming]
        E3[Final Output<br/>WAV File]
    end
    
    A1 --> B1
    A2 --> B2
    B2 --> B3
    
    B1 --> C1
    C1 --> C2
    C2 --> C3
    B3 --> C4
    
    C3 --> D1
    C4 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4
    
    D4 --> E1
    E1 --> E2
    E2 --> E3
    
    style Input fill:#e1f5ff
    style Preprocessing fill:#fff4e1
    style Encoding fill:#f0e1ff
    style Synthesis fill:#ffe1e1
    style Output fill:#e1ffe1
```

## 5.2 Detailed Design

### Module-Level Design

#### 5.2.1 Audio Preprocessing Module

**Purpose**: Prepare raw audio files for neural network processing by standardizing format, quality, and extracting relevant features.

**Class Design:**

```plantuml
@startuml
class AudioPreprocessor {
  - target_sample_rate: int = 24000
  - n_mels: int = 80
  - n_fft: int = 1024
  - hop_length: int = 256
  
  + __init__(config: dict)
  + load_audio(file_path: str): np.ndarray
  + resample_audio(audio: np.ndarray, orig_sr: int): np.ndarray
  + normalize_audio(audio: np.ndarray): np.ndarray
  + extract_mel_spectrogram(audio: np.ndarray): np.ndarray
  + trim_silence(audio: np.ndarray, threshold: float): np.ndarray
}

class FeatureExtractor {
  - mel_basis: np.ndarray
  - window: np.ndarray
  
  + compute_stft(audio: np.ndarray): np.ndarray
  + mel_filterbank(): np.ndarray
  + extract_mfcc(audio: np.ndarray): np.ndarray
}

class AudioValidator {
  + validate_format(file_path: str): bool
  + check_sample_rate(audio: np.ndarray): bool
  + check_duration(audio: np.ndarray, min_sec: float): bool
}

AudioPreprocessor --> FeatureExtractor
AudioPreprocessor --> AudioValidator
@enduml
```

**Algorithm: Silence Trimming**

```
FUNCTION trim_silence(audio_waveform, sample_rate, threshold=-45dB):
    INPUT: 
        audio_waveform: numpy array of audio samples
        sample_rate: integer (24000 Hz)
        threshold: float (silence threshold in dB)
    
    OUTPUT:
        trimmed_audio: numpy array without leading/trailing silence
    
    ALGORITHM:
        1. Convert audio to int16 format for processing
           audio_int16 ← (audio_waveform × 32767).astype(int16)
        
        2. Create AudioSegment object
           sound ← AudioSegment(data=audio_int16.bytes,
                                sample_width=2,
                                frame_rate=sample_rate,
                                channels=1)
        
        3. Split on silence
           chunks ← split_on_silence(sound,
                                      min_silence_len=100ms,
                                      silence_thresh=threshold,
                                      keep_silence=50ms)
        
        4. IF chunks is empty THEN
              RETURN single_zero_sample
           END IF
        
        5. Concatenate all non-silent chunks
           combined ← sum(chunks)
        
        6. Convert back to float32 normalized format
           samples ← array(combined.get_samples())
           trimmed_audio ← samples / 32767.0
        
        7. RETURN trimmed_audio
END FUNCTION
```

#### 5.2.2 Text Processing Module

**Purpose**: Convert input text into tokenized sequences suitable for neural network processing.

**Class Design:**

```plantuml
@startuml
class TextProcessor {
  - vocab: dict
  - vocab_size: int = 100
  - max_length: int = 1000
  
  + __init__(vocab_file: str)
  + tokenize(text: str): List[int]
  + detokenize(tokens: List[int]): str
  + pad_sequence(tokens: List[int], max_len: int): np.ndarray
  + clean_text(text: str): str
}

class CharacterTokenizer {
  - char_to_id: dict
  - id_to_char: dict
  
  + encode(text: str): List[int]
  + decode(token_ids: List[int]): str
  + build_vocabulary(corpus: List[str]): dict
}

class TextNormalizer {
  + expand_abbreviations(text: str): str
  + normalize_numbers(text: str): str
  + remove_special_chars(text: str): str
  + to_lowercase(text: str): str
}

TextProcessor --> CharacterTokenizer
TextProcessor --> TextNormalizer
@enduml
```

**Algorithm: Text Tokenization**

```
FUNCTION tokenize_text(input_text, vocabulary, max_length=1000):
    INPUT:
        input_text: string (raw text script)
        vocabulary: dictionary mapping characters to integers
        max_length: integer (maximum sequence length)
    
    OUTPUT:
        token_sequence: integer array of character IDs
    
    ALGORITHM:
        1. Normalize text
           cleaned_text ← clean_text(input_text)
           cleaned_text ← replace(cleaned_text, "'", "'")
           cleaned_text ← lowercase(cleaned_text)
        
        2. Initialize token list
           tokens ← empty list
        
        3. FOR each character c IN cleaned_text DO
              IF c IN vocabulary THEN
                  tokens.append(vocabulary[c])
              ELSE
                  tokens.append(vocabulary['<UNK>'])  // Unknown token
              END IF
           END FOR
        
        4. Truncate if necessary
           IF length(tokens) > max_length THEN
              tokens ← tokens[0:max_length]
           END IF
        
        5. Pad sequence to fixed length
           WHILE length(tokens) < max_length DO
              tokens.append(vocabulary['<PAD>'])
           END WHILE
        
        6. Convert to numpy array
           token_sequence ← np.array(tokens, dtype=int32)
        
        7. RETURN token_sequence
END FUNCTION
```

#### 5.2.3 Neural Model Inference Module

**Purpose**: Execute forward pass through encoder-decoder architecture to generate mel-spectrograms and synthesize audio.

**Class Design:**

```plantuml
@startuml
class ModelInference {
  - model: NeuralNetwork
  - device: str
  - inference_steps: int
  
  + __init__(model_path: str, device: str)
  + generate_speech(text_tokens: Tensor, speaker_emb: Tensor): Tensor
  + encode_text(tokens: Tensor): Tensor
  + decode_mel(encoded: Tensor, speaker: Tensor): Tensor
  + synthesize_audio(mel_spec: Tensor): np.ndarray
}

class Tacotron2Model {
  - encoder: TextEncoder
  - decoder: MelDecoder
  - postnet: ConvPostnet
  
  + forward(text: Tensor, speaker: Tensor): Tensor
  + inference(text: Tensor, speaker: Tensor): Tensor
}

class WaveGlowVocoder {
  - flows: List[CouplingLayer]
  - n_flows: int = 12
  
  + forward(mel: Tensor): Tensor
  + infer(mel: Tensor): Tensor
}

class SpeakerEncoder {
  - resnet: ResNet18
  - embedding_dim: int = 192
  
  + extract_embedding(mel: Tensor): Tensor
  + forward(audio: Tensor): Tensor
}

ModelInference --> Tacotron2Model
ModelInference --> WaveGlowVocoder
ModelInference --> SpeakerEncoder
@enduml
```

**Algorithm: Speech Generation**

```
FUNCTION generate_speech(text_input, reference_audio, cfg_scale=1.3):
    INPUT:
        text_input: string (text to synthesize)
        reference_audio: numpy array (voice sample)
        cfg_scale: float (classifier-free guidance scale)
    
    OUTPUT:
        audio_waveform: numpy array (synthesized speech)
    
    ALGORITHM:
        1. Preprocess inputs
           tokens ← tokenize(text_input)
           mel_ref ← extract_mel_spectrogram(reference_audio)
        
        2. Extract speaker embedding
           speaker_emb ← speaker_encoder(mel_ref)
           // speaker_emb shape: [192]
        
        3. Encode text sequence
           text_hidden ← text_encoder(tokens)
           // text_hidden shape: [seq_len, 512]
        
        4. Initialize decoder state
           decoder_hidden ← zeros([2, 1024])  // 2 layers
           attention_weights ← zeros([seq_len])
           mel_outputs ← empty list
        
        5. Autoregressive decoding
           previous_mel ← zeros([80])  // Start token
           
           FOR frame_idx FROM 0 TO max_frames DO
              // Apply attention
              context, attention_weights ← attention(
                  query=decoder_hidden,
                  keys=text_hidden,
                  prev_weights=attention_weights
              )
              
              // Concatenate inputs
              decoder_input ← concatenate([
                  previous_mel,
                  speaker_emb,
                  context
              ])
              
              // Decoder forward pass
              decoder_input ← prenet(decoder_input)
              decoder_output, decoder_hidden ← decoder_lstm(
                  decoder_input,
                  decoder_hidden
              )
              
              // Predict mel-frame
              mel_frame ← linear_projection(decoder_output)
              mel_outputs.append(mel_frame)
              
              // Update for next iteration
              previous_mel ← mel_frame
              
              // Check for stop condition
              stop_prob ← stop_token_predictor(decoder_output)
              IF stop_prob > 0.5 THEN
                  BREAK
              END IF
           END FOR
        
        6. Apply postnet refinement
           mel_spectrogram ← stack(mel_outputs)
           mel_spectrogram ← postnet(mel_spectrogram)
           // mel_spectrogram shape: [num_frames, 80]
        
        7. Apply classifier-free guidance (optional)
           IF cfg_scale > 1.0 THEN
              mel_uncond ← generate_unconditional(tokens)
              mel_spectrogram ← mel_uncond + cfg_scale × 
                               (mel_spectrogram - mel_uncond)
           END IF
        
        8. Vocoder synthesis
           audio_waveform ← waveglow_vocoder(mel_spectrogram)
           // audio_waveform shape: [num_samples]
        
        9. Post-process audio
           audio_waveform ← normalize(audio_waveform)
           audio_waveform ← trim_silence(audio_waveform)
        
        10. RETURN audio_waveform
END FUNCTION
```

### System Workflow Design

```mermaid
flowchart TD
    Start([User Initiates Generation]) --> Upload{Reference Audio<br/>Uploaded?}
    
    Upload -->|No| UploadPrompt[Prompt User to Upload Audio]
    UploadPrompt --> ProcessAudio[Process & Add to Voice Library]
    Upload -->|Yes| SelectVoice[Select Voice from Dropdown]
    ProcessAudio --> SelectVoice
    
    SelectVoice --> EnterText[Enter Text Script]
    EnterText --> SetParams[Set Generation Parameters<br/>CFG Scale, Silence Trim]
    
    SetParams --> Validate{Inputs Valid?}
    Validate -->|No| Error1[Display Error Message]
    Error1 --> EnterText
    
    Validate -->|Yes| StartGen[Initialize Generation]
    StartGen --> LoadModel[Load Neural Models<br/>Tacotron2 + WaveGlow]
    
    LoadModel --> ExtractEmb[Extract Speaker Embedding<br/>from Reference Audio]
    ExtractEmb --> EncodeText[Encode Text Sequence<br/>Character Tokenization]
    
    EncodeText --> AttentionLoop{Attention &<br/>Decoding Loop}
    AttentionLoop --> GenMel[Generate Mel-Spectrogram<br/>Frame by Frame]
    
    GenMel --> CheckStop{Stop Token<br/>Predicted?}
    CheckStop -->|No| AttentionLoop
    CheckStop -->|Yes| Postnet[Apply Postnet Refinement]
    
    Postnet --> Vocoder[WaveGlow Vocoding<br/>Mel → Audio Waveform]
    Vocoder --> PostProc[Post-Processing<br/>Normalization & Silence Trim]
    
    PostProc --> SaveFile[Save Audio File<br/>.wav Format]
    SaveFile --> GenMeta[Generate Metadata<br/>Timestamps JSON]
    
    GenMeta --> Display[Display Audio Player<br/>& Download Links]
    Display --> End([Generation Complete])
    
    style Start fill:#e1f5ff
    style End fill:#e1ffe1
    style Error1 fill:#ffe1e1
    style LoadModel fill:#fff4e1
    style Vocoder fill:#f0e1ff
```

### Database Schema Design

```plantuml
@startuml
entity "VoiceProfiles" as VP {
  * voice_id : VARCHAR(36) <<PK>>
  --
  voice_name : VARCHAR(100)
  audio_path : VARCHAR(255)
  embedding_vector : BLOB
  sample_rate : INTEGER
  duration_sec : FLOAT
  created_at : TIMESTAMP
  updated_at : TIMESTAMP
}

entity "GenerationHistory" as GH {
  * generation_id : VARCHAR(36) <<PK>>
  --
  voice_id : VARCHAR(36) <<FK>>
  input_text : TEXT
  output_path : VARCHAR(255)
  cfg_scale : FLOAT
  inference_steps : INTEGER
  duration_sec : FLOAT
  file_size_mb : FLOAT
  created_at : TIMESTAMP
}

entity "ModelCheckpoints" as MC {
  * checkpoint_id : VARCHAR(36) <<PK>>
  --
  model_name : VARCHAR(100)
  version : VARCHAR(20)
  file_path : VARCHAR(255)
  file_size_mb : FLOAT
  architecture : VARCHAR(50)
  parameters : JSON
  created_at : TIMESTAMP
}

entity "SystemLogs" as SL {
  * log_id : VARCHAR(36) <<PK>>
  --
  log_level : VARCHAR(20)
  message : TEXT
  module : VARCHAR(100)
  timestamp : TIMESTAMP
}

VP ||--o{ GH : "used in"
MC ||--o{ GH : "generated with"
@enduml
```

### Interface Design Specifications

**Gradio Interface Layout:**

```
┌─────────────────────────────────
