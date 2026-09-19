Authors: AI generated for Marlon Myd & Dirk Jan Buter 
Date: August 2026
Target Engine: Unity 3D Engine (2022.3 LTS+)
Target Distribution: Unity Asset Store Package & Cloud Infrastructure SDK
## 1. Product Vision & Conceptual Philosophy

### 1.1 The Core Mission

Project Persona Cloning bridges human identity and real-time 3D interaction. The objective is to give developers a plug-and-play Unity framework capable of reproducing living individuals, memorializing loved ones who have passed away, or bringing historical figures back to life.

Rather than building a basic chatbot attached to a 3D avatar, Persona Cloning treats human identity as a unified, three-part ecosystem:

  

1. Visual & Expressive Identity: How the person looks, ages, shifts weight, winks, or gestures during conversation.  
      
    
2. Vocal & Auditory Identity: How the person sounds, including tone, cadence, and real-time lip movements synced to low-latency voice streaming.  
      
    
3. Cognitive & Experiential Identity ("The Brains"): What the person remembers, how they speak, their core values, emotional triggers, and personality traits.  
      
    

### 1.2 Target Use Cases

- Self-Cloning (Legacy Archiving): Users create digital twins of themselves to serve as virtual assistants, legacy preservation tools, or interactive avatars in virtual reality.  
      
    
- Memorial Cloning (Loved Ones): Relatives assemble historical records, old audio recordings, letters, and memory questionnaires to build an interactive digital memorial of a deceased family member.  
      
    
- Historical & Public Figures: Developers assemble public domain text, archived speeches, and photos to reconstruct interactive historical personas (e.g., Albert Einstein, Socrates) for educational applications.  
      
    

## 2. Deep-Dive: How the System Architecture Works

The system operates across a Client-Server Hybrid Pipeline. High-fidelity rendering and motion tracking run locally on the client's GPU, while heavy artificial intelligence computations (RAG memory retrieval, vector processing, LLM generation, and TTS synthesis) are handled by scalable cloud backend services.

  

+-----------------------------------------------------------------------------------+

|                                 UNITY CLIENT ENGINE                               |

|                                                                                   |

|  +--------------------+    +--------------------+    +-------------------------+  |

|  | Avatar Creator     |    | MediaPipe MoCap    |    | Low-Latency Audio       |  |

|  | 'Womb' System      |    | Bridge (468 Mesh)  |    | Streaming Buffer        |  |

|  +---------+----------+    +---------+----------+    +------------+------------+  |

+------------|-------------------------|----------------------------|---------------+

             |                         |                            |

             | Asset/Mesh Config       | Real-Time Motion Vector    | WebSocket Stream

             v                         v                            v

+-----------------------------------------------------------------------------------+

|                                 BACKEND API SERVER                                |

|                                                                                   |

|  +--------------------+    +--------------------+    +-------------------------+  |

|  | Credit Metering &  |    | Vector RAG Store   |    | Cognitive State Engine  |  |

|  | Auth Middleware    |    | (Memory Engine)    |    | ("Brains")              |  |

|  +--------------------+    +--------------------+    +-------------------------+  |

+-----------------------------------------------------------------------------------+

  

## 3. Explanatory Module Deep-Dives & Practical Examples

### Module 1: Avatar Creator Engine ("Womb") & Shader Preservation

#### How It Works

The Avatar Creator "Womb" allows developers or end-users to generate customized base human models using parametric sliders (e.g., eye distance, jaw width, nose bridge height, age lines, body mass) or by projecting reference photos directly onto base dynamic meshes.

  

#### Technical Explanation & Common Problem

When dynamic 3D models (glTF/GLB format) are imported into Unity at runtime, Unity loses link references to material shaders. Standard materials often fallback to unlit black or magenta textures in standalone PC/Mac/Mobile builds.

  

To solve this, the Shader Fixer Routine iterates through every mesh renderer on a newly loaded dynamic avatar, re-attaching native Unity standard/URP shaders while mapping diffuse and normal maps back onto the material programmatically.

  

#### C# Example: Runtime Shader Rebinding

C#

using UnityEngine;

  

public class WombAvatarShaderFixer : MonoBehaviour

{

    [SerializeField] private Shader defaultTargetShader;

  

    /// <summary>

    /// Rebinds missing dynamic textures to native shaders at runtime when an avatar loads.

    /// </summary>

    public void RestoreAvatarMaterials(GameObject importedAvatar)

    {

        if (importedAvatar == null) return;

  

        // Fallback to Standard shader if none assigned

        Shader activeShader = defaultTargetShader != null 

            ? defaultTargetShader 

            : Shader.Find("Universal Render Pipeline/Lit") ?? Shader.Find("Standard");

  

        Renderer[] renderers = importedAvatar.GetComponentsInChildren<Renderer>(true);

  

        foreach (Renderer meshRenderer in renderers)

        {

            foreach (Material mat in meshRenderer.materials)

            {

                if (mat == null) continue;

  

                // Cache original textures before rebinding shader

                Texture baseTexture = mat.mainTexture 

                    ?? (mat.HasProperty("_BaseMap") ? mat.GetTexture("_BaseMap") : null)

                    ?? (mat.HasProperty("_MainTex") ? mat.GetTexture("_MainTex") : null);

  

                mat.shader = activeShader;

  

                // Re-assign cached textures into new shader properties

                if (baseTexture != null)

                {

                    if (mat.HasProperty("_BaseMap")) mat.SetTexture("_BaseMap", baseTexture);

                    if (mat.HasProperty("_MainTex")) mat.SetTexture("_MainTex", baseTexture);

                }

            }

        }

    }

}

  

### Module 2: The Cognitive Engine ("Brains") & Memory RAG Pipeline

#### How It Works

The "Brains" engine gives the persona a subjective memory, conversational identity, and distinct voice.

  

1. Dynamic Question Trees: The user completes a structured questionnaire covering childhood memories, core values, phrase preferences, and emotional traits.  
      
    
2. Vector Ingestion: The text and audio transcripts are broken into chunks, turned into vector embeddings, and saved in a vector store.  
      
    
3. Retrieval-Augmented Generation (RAG): When an end-user speaks to the clone, the server converts the input text into a vector, queries the memory database using cosine similarity, and pulls relevant past experiences into the prompt context.  
      
    

#### Concrete Context Construction Example

If an end-user asks the clone: "Do you remember where you went to school?"

  

$$\text{Cosine Similarity Score} = \frac{\mathbf{A} \cdot \mathbf{B}}{\Vert{}\mathbf{A}\Vert{} \Vert{}\mathbf{B}\Vert{}}$$

The system performs a vector lookup ($\text{Score} > 0.82$) and builds the following prompt payload sent to the language model:

  

JSON

{

  "system_instruction": "You are a clone of Dirk Jan. Maintain a witty, direct tone. Speak naturally with brief sentences.",

  "retrieved_memories": [

    "Attended primary school in Zwolle between 1998 and 2004.",

    "Built early C# game mods during secondary school computer lab sessions."

  ],

  "short_term_history": [

    {"role": "user", "content": "Hey Dirk, great to see you again."},

    {"role": "assistant", "content": "Good to see you too. What are we building today?"}

  ],

  "current_input": "Do you remember where you went to school?"

}

  

### Module 3: Low-Latency Streaming Audio & Voice Capture Pipeline

#### How It Works

Standard Unity web requests wait for an entire .wav or .mp3 file to download before generating an AudioClip. This causes 3–5 seconds of total delay, breaking conversational flow.

  

To achieve conversational response times ($< 1.5$ seconds), our system uses a chunked streaming audio buffer:

  

1. The backend streams raw 16-bit PCM voice bytes as sentence fragments via persistent WebSockets.  
      
    
2. The Unity C# audio manager queues incoming byte arrays directly into a dynamic circular float ring-buffer.  
      
    
3. Unity processes real-time audio sample output using OnAudioFilterRead, playing audio without waiting for the full response to generate.  
      
    

#### Audio Capture & Custom WAV Encoding Example

The microphone pipeline records audio input, tracks signal levels (RMS), trims silent audio, and formats raw byte arrays for server transmission:

  

C#

using System;

using System.IO;

using System.Text;

using UnityEngine;

  

public static class AudioEncodingPipeline

{

    /// <summary>

    /// Converts a recorded Unity AudioClip into a standard 16-bit PCM byte array.

    /// </summary>

    public static byte[] EncodeToWav(AudioClip clip)

    {

        if (clip == null) return null;

  

        float[] rawSamples = new float[clip.samples * clip.channels];

        clip.GetData(rawSamples, 0);

  

        int pcmByteCount = rawSamples.Length * 2; // 16-bit = 2 bytes per sample

  

        using (MemoryStream stream = new MemoryStream(44 + pcmByteCount))

        using (BinaryWriter writer = new BinaryWriter(stream))

        {

            // RIFF Chunk Descriptor

            writer.Write(Encoding.ASCII.GetBytes("RIFF"));

            writer.Write(36 + pcmByteCount);

            writer.Write(Encoding.ASCII.GetBytes("WAVE"));

  

            // fmt Sub-chunk

            writer.Write(Encoding.ASCII.GetBytes("fmt "));

            writer.Write(16); // Subchunk1Size for PCM

            writer.Write((short)1); // AudioFormat 1 = PCM

            writer.Write((short)clip.channels);

            writer.Write(clip.frequency);

            writer.Write(clip.frequency * clip.channels * 2); // Byte Rate

            writer.Write((short)(clip.channels * 2)); // Block Align

            writer.Write((short)16); // BitsPerSample

  

            // data Sub-chunk

            writer.Write(Encoding.ASCII.GetBytes("data"));

            writer.Write(pcmByteCount);

  

            // Convert floating point audio samples (-1.0 to 1.0) into 16-bit PCM short values

            for (int i = 0; i < rawSamples.Length; i++)

            {

                float clampedSample = Mathf.Clamp(rawSamples[i], -1.0f, 1.0f);

                short pcmValue = (short)(clampedSample * 32767f);

                writer.Write(pcmValue);

            }

  

            return stream.ToArray();

        }

    }

}

  

### Module 4: Real-Time Motion Capture via MediaPipe

#### How It Works

To keep human avatars expressive without requiring expensive motion capture suits:

  

1. Face Tracking: The MediaPipe Unity Bridge captures webcam video feed at 60 FPS, extracting 468 3D facial mesh points.  
      
    
2. Morph Target Translation: Point coordinates map directly onto 3D ARKit shape keys (e.g., mouth open, smile, eye blink, eyebrow raise).  
      
    
3. Upper Body Pose IK: MediaPipe Pose keypoints track shoulders, elbows, and wrists, mapping them to Unity Humanoid Skeleton Inverse Kinematics (IK) targets.  
      
    
4. Noise Filtering: Raw webcam tracking contains micro-jitter from lighting changes. Coordinates are passed through a One Euro Filter to maintain stable facial features and smooth movements.  
      
    

#### Landmark-to-Blendshape Mapping Matrix

|   |   |   |   |
|---|---|---|---|
|Facial Landmark Group|Extracted Feature Delta|ARKit Blendshape Target|Avatar Visual Result|
|Landmarks 13 & 14|Inner Lip Gap|jawOpen|Mouth opens naturally during speech|
|Landmarks 61 & 291|Corner Mouth Width|mouthSmileLeft / Right|Avatar grins when user smiles|
|Landmarks 159 & 145|Vertical Eyelid Distance|eyeBlinkLeft / Right|Natural blinking and winking|
|Landmarks 70 & 300|Brow Elevation Delta|browOuterUpLeft / Right|Expressive eyebrow movements|

## 4. Backend Infrastructure, Metering & Data Architecture

### 4.1 Usage Metering & Pricing Logic

Because the API processes intensive AI workloads (LLM generation, vector search, TTS rendering), backend services track usage through an API credit engine:

  

- LLM Tokens: Deducted based on input and output token volume.  
      
    
- Audio Processing: Billed per second of generated audio.  
      
    
- Storage Allotment: Vector store capacity and avatar file hosting billed per persona profile.
    

### 4.2 Database Relational Model (PostgreSQL)

SQL

-- User account & authentication table

CREATE TABLE users (

    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    email VARCHAR(255) UNIQUE NOT NULL,

    password_hash VARCHAR(255) NOT NULL,

    full_name VARCHAR(100) NOT NULL,

    subscription_tier VARCHAR(20) DEFAULT 'free',

    credit_balance NUMERIC(12, 4) DEFAULT 0.0000,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP

);

  

-- Consumption tracking for API operations

CREATE TABLE usage_logs (

    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID REFERENCES users(user_id),

    service_type VARCHAR(50) NOT NULL, -- 'TTS', 'STT', 'LLM', 'AVATAR_GEN'

    units_consumed NUMERIC(10, 2) NOT NULL,

    credits_deducted NUMERIC(10, 4) NOT NULL,

    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP

);

  

-- Stored Persona Configurations

CREATE TABLE persona_profiles (

    persona_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    owner_id UUID REFERENCES users(user_id),

    persona_type VARCHAR(20) NOT NULL, -- 'SELF', 'MEMORIAL', 'PUBLIC'

    display_name VARCHAR(100) NOT NULL,

    avatar_config_json JSONB NOT NULL,

    voice_profile_id VARCHAR(100),

    system_prompt_override TEXT,

    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP

);

  

## 5. Granular 16-Week Implementation Roadmap

Weeks  1-4  [Sprints 1-2]: Backend Schema, Stripe Billing, WebSockets Gateway

Weeks  5-8  [Sprints 3-4]: Avatar Creator "Womb", Runtime Mesh & Shader Repair

Weeks  9-12 [Sprints 5-6]: MediaPipe 468 Facial Mesh, ARKit Mapping & Pose IK

Weeks 13-14 [Sprint 7]   : Cognitive Engine "Brains", Vector RAG & Audio Streaming

Weeks 15-16 [Sprint 8]   : End-to-End Latency Tuning, Testing & Asset Store Release

  

### Granular Task Breakdown

Phase 1: Backend Infrastructure & Billing (Sprints 1-2 | Lead: Dirk Jan)

- API-101: PostgreSQL schema implementation for user accounts and credit ledgers.

- API-102: Usage-based credit metering engine for tracking LLM tokens and audio processing.

- API-103: Stripe webhook payment integration for automatic subscription top-ups.

- API-104: Persistent bi-directional WebSocket gateway for streaming connections.

  

Phase 2: Avatar Creator Engine "Womb" (Sprints 3-4 | Lead: Dirk Jan & Marlon)

- WMB-201: Dynamic mesh generation tools ported for Unity runtime compatibility.

- WMB-202: 3D UI interface sliders for controlling dynamic facial parameters.

- WMB-203: Runtime shader fallback and texture restoration scripts.

- WMB-204: GLB and AssetBundle serialization pipelines for cloud syncing.

  

Phase 3: MediaPipe Motion Capture (Sprints 5-6 | Lead: Marlon Myd)

- MOT-301: Native C# MediaPipe Unity bridge integration.

- MOT-302: Mapping 468 facial mesh landmarks to ARKit morph target blendshapes.

- MOT-303: One Euro temporal filtering for smooth motion tracking.

- MOT-304: Upper-body pose keypoint mapping to Unity Humanoid Skeleton IK.

  

Phase 4: Cognitive Memory Engine "Brains" (Sprint 7 | Lead: Dirk Jan)

- BRN-401: Dynamic branching questionnaire sequences for capturing personal history.

- BRN-402: Vector database ingestion pipeline for text and transcript embedding.

- BRN-403: Dynamic context-window merger for short-term and long-term memory synthesis.

- BRN-404: Low-latency streaming audio ring-buffer for stutter-free audio output.

  

Phase 5: System Verification & Release (Sprint 8 | Joint Team)

- INT-501: Complete validation of Self, Memorial, and Public figure workflows.

- INT-502: Latency benchmarking targeting < 1.5 seconds round-trip response time.

- INT-503: Final packaging and documentation for Unity Asset Store submission.

  

## 6. Verification & System Quality Gates

|   |   |   |
|---|---|---|
|Quality Metric|Performance Target|Verification Method|
|Round-Trip Voice Latency|$< 1,500\,\text{ms}$ total|Microsecond timer from speech stop (Microphone.End) to initial audio output in Unity.|
|Client Rendering|$\ge 60\,\text{FPS}$ at 1080p|Unity Profiler running MediaPipe face/pose tracking alongside dynamic avatar rendering.|
|Tracking Jitter|$< 2\,\text{px}$ position delta|MediaPipe landmark stability verification under low-light ($< 50\,\text{lux}$) conditions.|
|Memory Retrieval Accuracy|$> 92\%$ relevance|RAG retrieval evaluation against questionnaire ground-truth data vectors.|
|Billing Accuracy|$100\%$ precision|Audit comparing token and audio API usage logs against Stripe account deductions.|