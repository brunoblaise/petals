<p align="center">
    <img src="https://i.imgur.com/7eR7Pan.png" width="400"><br>
    Run 100B+ language models at home, BitTorrent-style.<br>
    Fine-tuning and inference <a href="https://github.com/bigscience-workshop/petals#benchmarks">up to 10x faster</a> than offloading<br><br>
    <a href="https://pypi.org/project/petals/"><img src="https://img.shields.io/pypi/v/petals.svg?color=green"></a><br>
</p>

Generate text using distributed 176B-parameter [BLOOM](https://huggingface.co/bigscience/bloom) or [BLOOMZ](https://huggingface.co/bigscience/bloomz) and fine-tune them for your own tasks:

```python
from petals import DistributedBloomForCausalLM

model = DistributedBloomForCausalLM.from_pretrained("bigscience/bloom-petals", tuning_mode="ptune", pre_seq_len=16)
# Embeddings & prompts are on your device, BLOOM blocks are distributed across the Internet

inputs = tokenizer("A cat sat", return_tensors="pt")["input_ids"]
outputs = model.generate(inputs, max_new_tokens=5)
print(tokenizer.decode(outputs[0]))  # A cat sat on a mat...

# Fine-tuning (updates only prompts or adapters hosted locally)
optimizer = torch.optim.AdamW(model.parameters())
for input_ids, labels in data_loader:
    outputs = model.forward(input_ids)
    loss = cross_entropy(outputs.logits, labels)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

<p align="center">
    🚀 &nbsp;<b><a href="https://colab.research.google.com/drive/1Ervk6HPNS6AYVr3xVdQnY5a-TjjmLCdQ?usp=sharing">Try now in Colab</a></b>
</p>

🔏 Your data will be processed by other people in the public swarm. Learn more about privacy [here](https://github.com/bigscience-workshop/petals/wiki/Security,-privacy,-and-AI-safety). For sensitive data, you can set up a [private swarm](https://github.com/bigscience-workshop/petals/wiki/Launch-your-own-swarm) among people you trust.

### Connect your GPU and increase Petals capacity

Run this in an [Anaconda](https://www.anaconda.com) env (requires Linux and Python 3.7+):

```bash
conda install pytorch pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -U petals
python -m petals.cli.run_server bigscience/bloom-petals
```

Or use our [Docker](https://www.docker.com) image (works on Linux, macOS, and Windows with [WSL2](https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl)):

```bash
sudo docker run -p 31330:31330 --ipc host --gpus all --volume petals-cache:/cache --rm \
    learningathome/petals:main python -m petals.cli.run_server bigscience/bloom-petals --port 31330
```

📚 See [FAQ](https://github.com/bigscience-workshop/petals/wiki/FAQ:-Frequently-asked-questions#running-a-server) to learn how to configure the server to use multiple GPUs, address common issues, etc.

You can also host [BLOOMZ](https://huggingface.co/bigscience/bloomz), a version of BLOOM fine-tuned to follow human instructions in the zero-shot regime — just replace `bloom-petals` with `bloomz-petals`.

🔒 Hosting a server does not allow others to run custom code on your computer. Learn more about security [here](https://github.com/bigscience-workshop/petals/wiki/Security,-privacy,-and-AI-safety).

💬 If you have any issues or feedback, let us know on [our Discord server](https://discord.gg/D9MwApKgWa)!

### Check out tutorials, examples, and more

Basic tutorials:

- Getting started: [tutorial](https://colab.research.google.com/drive/1Ervk6HPNS6AYVr3xVdQnY5a-TjjmLCdQ?usp=sharing)
- Prompt-tune BLOOM to create a personified chatbot: [tutorial](https://colab.research.google.com/github/bigscience-workshop/petals/blob/main/examples/prompt-tuning-personachat.ipynb)
- Prompt-tune BLOOM for text semantic classification: [tutorial](https://colab.research.google.com/github/bigscience-workshop/petals/blob/main/examples/prompt-tuning-sst2.ipynb)

Useful tools and advanced guides:

- [Chatbot web app](http://chat.petals.ml) (connects to Petals via an HTTP endpoint): [source code](https://github.com/borzunov/chat.petals.ml)
- [Monitor](http://health.petals.ml) for the public swarm: [source code](https://github.com/borzunov/health.petals.ml)
- Launch your own swarm: [guide](https://github.com/bigscience-workshop/petals/wiki/Launch-your-own-swarm)
- Run a custom foundation model: [guide](https://github.com/bigscience-workshop/petals/wiki/Run-a-custom-model-with-Petals)

Learning more:

- Frequently asked questions: [FAQ](https://github.com/bigscience-workshop/petals/wiki/FAQ:-Frequently-asked-questions)
- In-depth system description: [paper](https://arxiv.org/abs/2209.01188)

📋 If you build an app running BLOOM with Petals, make sure it follows the BLOOM's [terms of use](https://huggingface.co/bigscience/bloom).

## How does it work?

- Petals runs large language models like [BLOOM-176B](https://huggingface.co/bigscience/bloom) **collaboratively** — you load a small part of the model, then team up with people serving the other parts to run inference or fine-tuning.
- Single-batch inference runs at ≈ 1 sec per step (token) — [up to 10x faster](https://github.com/bigscience-workshop/petals#benchmarks) than offloading, enough for [chatbots](http://chat.petals.ml) and other interactive apps. Parallel inference reaches hundreds of tokens/sec.
- Beyond classic language model APIs — you can employ any fine-tuning and sampling methods, execute custom paths through the model, or see its hidden states. You get the comforts of an API with the flexibility of PyTorch.

<p align="center">
    <img src="https://i.imgur.com/RTYF3yW.png" width="800">
</p>

<p align="center">
    📚 &nbsp;<b><a href="https://github.com/bigscience-workshop/petals/wiki/FAQ:-Frequently-asked-questions">See FAQ</a></b>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    📜 &nbsp;<b><a href="https://arxiv.org/pdf/2209.01188.pdf">Read paper</a></b>
</p>

## Installation

Here's how to install Petals with [Anaconda](https://www.anaconda.com/products/distribution) on Linux:

```bash
conda install pytorch pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -U petals
```

If you don't use Anaconda, you can install PyTorch in [any other way](https://pytorch.org/get-started/locally/). If you want to run models with 8-bit weights, please install PyTorch with CUDA 11.x or newer for compatility with [bitsandbytes](https://github.com/timDettmers/bitsandbytes).

See the instructions for macOS and Windows, the full requirements, and troubleshooting advice in our [FAQ](https://github.com/bigscience-workshop/petals/wiki/FAQ:-Frequently-asked-questions#running-a-client).

## Benchmarks

<table align="center">
  <tr>
    <th colspan="2">Network</th>
    <th colspan="2">Single-batch inference<br>(steps/s)</th>
    <th colspan="2">Parallel forward<br>(tokens/s)</th>
  </tr>
  <tr>
    <th rowspan="2">Bandwidth</th>
    <th rowspan="2">Round-trip<br>latency</th>
    <th colspan="2">Sequence length</th>
    <th colspan="2">Batch size</th>
  </tr>
  <tr align="center">
    <td>128</td>
    <td>2048</td>
    <td>1</td>
    <td>64</td>
  </tr>
  <tr>
    <th colspan="6">Offloading, max. possible speed on 1x A100 <sup>1</sup></th>
  </tr>
  <tr align="center">
    <td>256 Gbit/s</td>
    <td></td>
    <td>0.18</td>
    <td>0.18</td>
    <td>2.7</td>
    <td>170.3</td>
  </tr>
  <tr align="center">
    <td>128 Gbit/s</td>
    <td></td>
    <td>0.09</td>
    <td>0.09</td>
    <td>2.4</td>
    <td>152.8</td>
  </tr>
  <tr>
    <th colspan="6">Petals on 14 heterogeneous servers across Europe and North America <sup>2</sup></th>
  </tr>
  <tr align="center">
    <td colspan="2">Real world</td>
    <td>0.83</td>
    <td>0.79</td>
    <td>32.6</td>
    <td>179.4</td>
  </tr>
  <tr>
    <th colspan="6">Petals on 3 servers, with one A100 each <sup>3</sup></th>
  </tr>
  <tr align="center">
    <td>1 Gbit/s</td>
    <td>&lt; 5 ms</td>
    <td>1.71</td>
    <td>1.54</td>
    <td>70.0</td>
    <td>253.6</td>
  </tr>
  <tr align="center">
    <td>100 Mbit/s</td>
    <td>&lt; 5 ms</td>
    <td>1.66</td>
    <td>1.49</td>
    <td>56.4</td>
    <td>182.0</td>
  </tr>
  <tr align="center">
    <td>100 Mbit/s</td>
    <td>100 ms</td>
    <td>1.23</td>
    <td>1.11</td>
    <td>19.7</td>
    <td>112.2</td>
  </tr>
</table>

<sup>1</sup> **An upper bound for offloading performance.** We base our offloading numbers on the best possible hardware setup for offloading: CPU RAM offloading via PCIe 4.0 with 16 PCIe lanes per GPU and PCIe switches for pairs of GPUs. We assume zero latency for the upper bound estimation. In 8-bit, the model uses 1 GB of memory per billion parameters. PCIe 4.0 with 16 lanes has a throughput of 256 Gbit/s, so offloading 176B parameters takes 5.5 seconds. The throughput is twice as slow (128 Gbit/s) if we have two GPUs behind the same PCIe switch.

<sup>2</sup> **A real-world distributed setting** with 14 servers holding 2× RTX 3060, 4× 2080Ti, 2× 3090, 2× A4000, and 4× A5000 GPUs. These are personal servers and servers from university labs, spread across Europe and North America and connected to the Internet at speeds of 100–1000 Mbit/s. 4 servers operate from under firewalls.

<sup>3</sup> **An optimistic setup** that requires least communication. The client nodes have 8 CPU cores and no GPU.

We provide more evaluations and discuss these results in more detail in **Section 3.3** of our [paper](https://arxiv.org/pdf/2209.01188.pdf).

## 🛠️ Contributing

Please see our [FAQ](https://github.com/bigscience-workshop/petals/wiki/FAQ:-Frequently-asked-questions#contributing) on contributing.

## 📜 Citation

Alexander Borzunov, Dmitry Baranchuk, Tim Dettmers, Max Ryabinin, Younes Belkada, Artem Chumachenko, Pavel Samygin, and Colin Raffel.
[Petals: Collaborative Inference and Fine-tuning of Large Models.](https://arxiv.org/abs/2209.01188)
_arXiv preprint arXiv:2209.01188,_ 2022.

```bibtex
@article{borzunov2022petals,
  title = {Petals: Collaborative Inference and Fine-tuning of Large Models},
  author = {Borzunov, Alexander and Baranchuk, Dmitry and Dettmers, Tim and Ryabinin, Max and Belkada, Younes and Chumachenko, Artem and Samygin, Pavel and Raffel, Colin},
  journal = {arXiv preprint arXiv:2209.01188},
  year = {2022},
  url = {https://arxiv.org/abs/2209.01188}
}
```

--------------------------------------------------------------------------------

<p align="center">
    This project is a part of the <a href="https://bigscience.huggingface.co/">BigScience</a> research workshop.
</p>
<p align="center">
    <img src="https://petals.ml/bigscience.png" width="150">
</p>
