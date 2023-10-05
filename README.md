# text-to-image


{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "private_outputs": true,
      "provenance": [],
      "include_colab_link": true
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    },
    "accelerator": "GPU",
    "gpuClass": "standard"
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/gist/AlphaCodeHub/31e8734b9e8323dfa2c7edf777e798ee/text-to-image.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "code",
      "source": [
        "%pip install --quiet --upgrade diffusers transformers accelerate invisible_watermark mediapy"
      ],
      "metadata": {
        "id": "ufD_d64nr08H"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "use_refiner = False"
      ],
      "metadata": {
        "id": "hRVsA7iYxpMj"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "import mediapy as media\n",
        "import random\n",
        "import sys\n",
        "import torch\n",
        "\n",
        "from diffusers import DiffusionPipeline\n",
        "\n",
        "pipe = DiffusionPipeline.from_pretrained(\n",
        "    \"stabilityai/stable-diffusion-xl-base-1.0\",\n",
        "    torch_dtype=torch.float16,\n",
        "    use_safetensors=True,\n",
        "    variant=\"fp16\",\n",
        "    )\n",
        "\n",
        "if use_refiner:\n",
        "  refiner = DiffusionPipeline.from_pretrained(\n",
        "      \"stabilityai/stable-diffusion-xl-refiner-1.0\",\n",
        "      text_encoder_2=pipe.text_encoder_2,\n",
        "      vae=pipe.vae,\n",
        "      torch_dtype=torch.float16,\n",
        "      use_safetensors=True,\n",
        "      variant=\"fp16\",\n",
        "  )\n",
        "\n",
        "  refiner = refiner.to(\"cuda\")\n",
        "\n",
        "  pipe.enable_model_cpu_offload()\n",
        "else:\n",
        "  pipe = pipe.to(\"cuda\")"
      ],
      "metadata": {
        "id": "bG2hkmSEvByV"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "code",
      "source": [
        "prompt = \"imran khan win election 2023\"\n",
        "seed = random.randint(0, sys.maxsize)\n",
        "\n",
        "images = pipe(\n",
        "    prompt = prompt,\n",
        "    output_type = \"latent\" if use_refiner else \"pil\",\n",
        "    generator = torch.Generator(\"cuda\").manual_seed(seed),\n",
        "    ).images\n",
        "\n",
        "if use_refiner:\n",
        "  images = refiner(\n",
        "      prompt = prompt,\n",
        "      image = images,\n",
        "      ).images\n",
        "\n",
        "print(f\"Prompt:\\t{prompt}\\nSeed:\\t{seed}\")\n",
        "media.show_images(images)\n",
        "images[0].save(\"output.jpg\")"
      ],
      "metadata": {
        "id": "AUc4QJfE-uR9"
      },
      "execution_count": null,
      "outputs": []
    }
  ]
}
