![Google Drive](https://img.shields.io/badge/Google%20Drive-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)![NumPy](https://img.shields.io/badge/numpy-%23013243.svg?style=for-the-badge&logo=numpy&logoColor=white)![PyTorch](https://img.shields.io/badge/PyTorch-%23EE4C2C.svg?style=for-the-badge&logo=PyTorch&logoColor=white)![HuggingFace](https://img.shields.io/badge/huggingface-%23FFD21E.svg?style=for-the-badge&logo=huggingface&logoColor=white)![Google Colab](https://img.shields.io/badge/Google%20Colab-%23F9A825.svg?style=for-the-badge&logo=googlecolab&logoColor=white)![LoRA](https://img.shields.io/badge/LoRA-green?style=for-the-badge)

# LoRA SD Generator

[![ru](https://img.shields.io/badge/README_на_русском-2A2C39?style=for-the-badge&logo=github&logoColor=white)](README.ru.md)  

**LoRA-SD-Generator** is a personal research project on fine-tuning (**fine-tuning**) the **Stable Diffusion 1.5** model using the **LoRA** (**Low-Rank Adaptation**) method to generate stylized faces from text descriptions.

_The goal of the project_ is to create a lightweight, efficient, and reproducible model that can generate high-quality stylized portraits while maintaining low resource consumption.


## A bit of theory

To fully understand the project, let's explore the underlying technologies:

-  **Stable Diffusion (SD)**: A latent diffusion model from **Stability AI** trained on billions of images (**LAION-5B**). It operates in a latent space (compressed using **VAE**), where noise is gradually removed to generate images.

- **LoRA (Low-Rank Adaptation)**: **Microsoft's** **PEFT** method that adapts large models (like **SD**) by adding low-rank matrices to the weights. Instead of retraining all ~860M parameters of **SD**, **LoRA** only trains ~1-10M (`rank=16` in the project). This reduces **VRAM** to 4-6GB and speeds up training.

- **DreamBooth**: This is a simple and powerful way to fine-tune **Stable Diffusion** so that the model learns to draw a specific person, animal, object, or style well from 3-5 of your photos.

- **BLIP (Bootstrapped Language-Image Pre-training)**: A model from **Salesforce** for generating captions for images. The **BLIP** project (`Salesforce/blip-image-captioning-base`) creates text descriptions for the CelebA dataset.

## Key features:

> You only need to run the code with a GPU! In my example,
> I use **T4**.

- Automatic data preparation: loading, preprocessing (resize 512x512), and captioning using **BLIP**;

- Training using the `train_dreambooth_lora.py` script from **Hugging Face Diffusers**;

- Integration with **Google Drive** for storing dataset, models and generated images;

- Optimization for **T4 GPU**: mixed precision (fp16), gradient accumulation.

**Result**: The **LoRA** model (`pytorch_lora_weights.safetensors`), capable of generating stylized faces based on text suggestions.

> Also, to work with the code, you will need a **HuggingFace** token and allow
> access to your **Google Drive**. More detailed step-by-step instructions
> presented in the work itself!

## Metrics

### What metrics are used?

-  **FID (Frechet Inception Distance)**: Estimates how close the distribution of generated images is to the distribution of real images in the feature space of **InceptionV3**. Vector representations of real and synthetic images are taken, their means and covariances are calculated, and then the **Frechet distance** between the two Gaussian distributions is computed. Lower is better (0 = ideal).

- **CLIP score (Contrastive Language-Image Pre-training)**: This is usually the cosine similarity between the embedding of the text (hint/signature) and the embedding of the image, calculated using the **CLIP** model. The value is in the range of approximately [-1, 1]; higher is the stronger correspondence between the image and the text. It is often averaged over a variety of text–image pairs.

### What are the metrics and how can they be improved?

FID = 149.66
CLIP = 0.28

**Methods to improve metric performance:**

- Increase the size of the dataset or use reasonable augmentations;

- Increase the number of training steps or train for a longer time with a reduced learning rate;

- Increase the rank (LoRA);

- Increase the batch size if the GPU allows it.

## 📸 Example of code operation

### Main interface

The screenshots below show the main workflow: setting up prompt parameters and the generated image.

<p  align="center">

<img

width="1280"

height="433"

src="https://github.com/user-attachments/assets/730407ac-ec75-4f20-9db2-e6f8f0fc1738"

>

<br>

</p>

  

---

  

<p  align="center">

<img

width="1280"

height="407"

src="https://github.com/user-attachments/assets/03d57d8d-1cc3-495d-8404-922b7101c4a3"

>

<br>

</p>

### What are the settings for the prompt parameters?

1.  **Prompt** is a text description of what you want to see in the image (the main instruction for the model);

2.  **Unwanted elements** are a negative prompt: a list of what you do NOT want to see. The model actively tries to avoid them;

3.  **Random prompt** — a button that automatically fills the "Prompt" field with a random description from your dataset;

4.  **Steps** — the number of noise cleaning iterations (20–150). Less is faster but lower quality; more is more detailed but longer. Optimal is 50–100;

5.  **Prompt Follow Strength** — Guidance scale (1.0–20.0). The higher you go, the more accurately the model follows your text, but there may be overkill (artifacts). 7.5 — classic balance;

6. **Degree of conversion** — Strength (0.1–1.0) for img2img mode. Shows how much the model will change the original image. 0.1 — minimal edits, 1.0 — almost complete regeneration;

7. **Source image** is a field for uploading an image (if you want to use img2img). The model takes this image as a basis and stylizes it according to the prompt. If you do not upload an image, the model will generate it from scratch.

These parameters give you complete control, from simple text input to complex styling of your photos.