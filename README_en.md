# Stable Diffusion API to OpenAI API

(This file was translated into English by AI)

[Chinese](https://github.com/shulinbao/stable-diffusion-api-to-openai)

This project converts the image generation API format of [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) (`/sdapi/v1/txt2img`) into OpenAI's universal API format (`/v1/images/generations`), allowing it to relay requests to models like DALL-E 3.

To use this project, you first need an operational [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) server.

This project can also be used with [songquanpeng/one-api](https://github.com/songquanpeng/one-api) or [Calcium-Ion/new-api](https://github.com/Calcium-Ion/new-api), as these do not currently support the [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) API format. Additionally, you can use this project for advanced scenarios, such as **simulating DALL-E 3 using Stable Diffusion models**.

## Development Progress

- [x] Image generation
  - [x] Prompts
  - [x] Models
  - [x] Image size
  - [ ] Image quality ("hd or standard")
  - [x] Response format *(partial URL support)*
  - [x] Quantity
  - [ ] Styles ("vivid or natural")
  - [ ] Users
- [ ] Image editing
- [ ] Variations

## Model Configuration

The `config` directory includes templates for translating model parameters. You can customize the parameters for redirecting requests to Stable Diffusion Web UI when users select `dall-e-1`, `dall-e-2`, or `dall-e-3` models:

```json
{
  "sampler_name": "DPM++ 2S a Karras",
  "refiner_checkpoint": "sd_xl_refiner_1.0.safetensors",
  "refiner_switch_at": 0.8,
  "steps": 40,
  "cfg_scale": 7.0,
  "hr_upscaler": "Latent",
  "denoising_strength": 0.68,
  "subseed_strength": 0.75,
  "restore_faces": false,
  "negative_prompt": "ac_neg1 ac_neg2",
  "override_settings_restore_afterwards": false,
  "override_settings": {
    "sd_model_checkpoint": "opendalle_v11.safetensors",
    "sd_vae": "fixFP16ErrorsSDXLLowerMemoryUse_v10.safetensors"
  }
}
```

**Before use, modify the template with values applicable to your Stable Diffusion Web UI instance. If you want to retain default values from Stable Diffusion Web UI, you can omit them.**

## Deployment Instructions

We recommend deploying with `docker-compose`:

```bash
git clone https://github.com/shulinbao/stable-diffusion-api-to-openai
cd stable-diffusion-api-to-openai
```

Before deployment, update the following values in the `docker-compose.yml` file (see comments for guidance):

```yaml
services:
  server:
    build:
      context: .
    tty: true
    image: ghcr.io/matatonic/openedai-images
    environment:
      - SD_BASE_URL=http://stable-diffusion-webui:3001      # Replace with your Stable Diffusion Web UI URL
    volumes:
      - ./config:/app/config
    ports:
      - 5005:5005          # You can change this
    command: ["python", "images.py", "--host", "0.0.0.0", "--port", "5005"]
```

Then execute:

```bash
docker-compose up --build -d
```

Check the server logs to verify the status:

```bash
docker logs stable-diffusion-api-to-openai_server_1  # Or the name you customized
```

## Testing the API

You can test the API using OpenAI's format:

```bash
curl https://your-server-url/v1/images/generations \     # Replace with your server URL
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "dall-e-3",
    "prompt": "A cute baby sea otter",
    "n": 1,
    "size": "1024x1024"
  }'
```

## Notes

- `songquanpeng/one-api` and `Calcium-Ion/new-api` do not support custom image sizes for DALL-E 2 or DALL-E 3. To resolve this, modify the following section in `relay/relay-image.go`:

```go
	if imageRequest.Model == "dall-e-2" || imageRequest.Model == "dall-e" {
		if imageRequest.Size != "" && imageRequest.Size != "256x256" && imageRequest.Size != "512x512" && imageRequest.Size != "1024x1024" {     // DALL-E image size validation
			return nil, errors.New("size must be one of 256x256, 512x512, or 1024x1024, dall-e-3 1024x1792 or 1792x1024")
		}
	} else if imageRequest.Model == "dall-e-3" {
		if imageRequest.Size != "" && imageRequest.Size != "1024x1024" && imageRequest.Size != "1024x1792" && imageRequest.Size != "1792x1024" {
			return nil, errors.New("size must be one of 256x256, 512x512, or 1024x1024, dall-e-3 1024x1792 or 1792x1024")    // DALL-E 3 image size validation
		}
		if imageRequest.Quality == "" {
			imageRequest.Quality = "standard"
		}
	}
```

Add your required image sizes, then redeploy your `songquanpeng/one-api` or `Calcium-Ion/new-api` service.

## Acknowledgments

- **Matatonic**, for the original development of this project. 
