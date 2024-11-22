Stable Diffusion API to OpenAI API
---------------

[English](https://github.com/shulinbao/stable-diffusion-api-to-openai/blob/main/README_en.md)

这个项目可以把 [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 格式的图片生成 API `/sdapi/v1/txt2img` 转发为 OpenAI 的通用 API 格式 `/v1/images/generations`，并转发为 dall-e-3 等指定模型。

使用这个项目，您首先需要有一个 [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 的服务端。

您可将本项目搭配 [songquanpeng/one-api](https://github.com/songquanpeng/one-api) 或 [Calcium-Ion/new-api](https://github.com/Calcium-Ion/new-api) 使用，因为它们目前不支持 [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) 格式的 API。您还可以用它实现些高级的玩法，比如~~用 Stable Diffusion 模型伪装 dall-e-3~~。

## 开发进度

- [x] 图片生成
- - [x] 提示词
- - [x] 模型
- - [x] 图片大小
- - [ ] 图片质量 ("hd or standard")
- - [x] 响应格式 *(部分支持 url)
- - [x] 数量
- - [ ] 风格 ("vivid or natural")
- - [ ] 用户
- [ ] 图片编辑
- [ ] 变化

## 模型说明

`config` 目录预置了模型翻译的模板。您可以定制用户使用 `dall-e-1` `dall-e-2` `dall-e-3` 三个模型时所转接的 Stable Diffusion Web UI 参数：

```
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
**请在使用前把模板中的内容改为您 Stable Diffusion Web UI 中可用的参数，如果有的值想保持 sdwebui 的默认值，可以删掉**

## 部署方法

我们建议您使用 `docker-compose` 部署

```
git clone https://github.com/shulinbao/stable-diffusion-api-to-openai
cd stable-diffusion-api-to-openai
```

在部署之前，请记得修改 `` 文件的以下内容（参见注释）

```
services:
  server:
    build:
      context: .
    tty: true
    image: ghcr.io/matatonic/openedai-images
    environment:
      - SD_BASE_URL=http://stable-diffusion-webui:3001      # Please change it to your sdwebui url
    volumes:
      - ./config:/app/config
    ports:
      - 5005:5005          # You could change it
    command: ["python", "images.py", "--host", "0.0.0.0", "--port", "5005"]
```

然后

```
docker-compose up --build -d
```

检查运行状态

```
docker logs stable-diffusion-api-to-openai_server_1  # 或者你自定义的名称
```

## 测试 API

您可以按照 OpenAI API 的格式使用 API 并进行测试：

```
curl https://your-server-url/v1/images/generations \     # 将 url 更换为你部署的服务器的
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "dall-e-3",
    "prompt": "A cute baby sea otter",
    "n": 1,
    "size": "1024x1024"
  }'
```

## 注意事项

- songquanpeng/one-api 还有 Calcium-Ion/new-api 不支持自定义大小的 dall-e-2 或 dall-e-3 图片。为了解决这个问题，您需要修改 `relay/relay-image.go` 中的以下内容：

```
	if imageRequest.Model == "dall-e-2" || imageRequest.Model == "dall-e" {
		if imageRequest.Size != "" && imageRequest.Size != "256x256" && imageRequest.Size != "512x512" && imageRequest.Size != "1024x1024" {     // dall-e 的图片大小判定
			return nil, errors.New("size must be one of 256x256, 512x512, or 1024x1024, dall-e-3 1024x1792 or 1792x1024")
		}
	} else if imageRequest.Model == "dall-e-3" {
		if imageRequest.Size != "" && imageRequest.Size != "1024x1024" && imageRequest.Size != "1024x1792" && imageRequest.Size != "1792x1024" {
			return nil, errors.New("size must be one of 256x256, 512x512, or 1024x1024, dall-e-3 1024x1792 or 1792x1024")    // dall-e-3 的图片大小判定
		}
		if imageRequest.Quality == "" {
			imageRequest.Quality = "standard"
		}
	}
```
添加你需要的图片大小，然后重新部署你的 songquanpeng/one-api 或 Calcium-Ion/new-api 即可。

## 致谢
- matatonic，本项目最早由其开发
