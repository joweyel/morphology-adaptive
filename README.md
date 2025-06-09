# morphology-adaptive

This repository demonstrates how a single locomotion controller can work with different shapes (biped and quadruped). It uses an attention mechanism to handle an arbitrary number of inputs, and the same module is shared across all muscles to support an arbitrary number of outputs.

**More details will be available in an upcoming paper.**

![](media/anim.gif)

For simulation, this repository uses [Algovivo](https://github.com/juniorrojas/algovivo), originally built for the browser using WebAssembly, but here a native build is used to enable PyTorch integration.

The workflow [`trajectory-attn.yml`](.github/workflows/trajectory-attn.yml) runs the controller on both morphologies and generates a video. If you have your own copy or fork of this repository, you can [run the workflow from the GitHub Actions UI](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow), no local installation needed. Once it completes, the video will be saved as a workflow artifact and can be downloaded from the workflow run page.


# morphology-adaptive

This repository demonstrates how a single locomotion controller can work with different shapes (biped and quadruped). It uses an attention mechanism to handle an arbitrary number of inputs, and the same module is shared across all muscles to support an arbitrary number of outputs.

**More details will be available in an upcoming paper.**

![](media/anim.gif)

For simulation, this repository uses [Algovivo](https://github.com/juniorrojas/algovivo), originally built for the browser using WebAssembly, but here a native build is used to enable PyTorch integration.

The workflow [`trajectory-attn.yml`](.github/workflows/trajectory-attn.yml) runs the controller on both morphologies and generates a video. If you have your own copy or fork of this repository, you can [run the workflow from the GitHub Actions UI](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/manually-running-a-workflow), no local installation needed. Once it completes, the video will be saved as a workflow artifact and can be downloaded from the workflow run page.



## Running the Program locally

### 1. **Clone repositories**
```bash
# git clone --branch main https://github.com/joweyel/morphology-adaptive
git clone https://github.com/juniorrojas/algovivo.git algovivo.repo
git clone https://github.com/juniorrojas/algovivo.git algovivo.build.repo
```
Checkout specific commits:
```bash
cd algovivo.repo && git checkout 4d3abd72c43d9d680d1514d9b24b042ef9e46a8f && cd ..
cd algovivo.build.repo && git checkout 02041c91eb67142fe1a08e10944e214a872d44db && cd ..
```
Copy required build files:
```bash
cp -r algovivo.build.repo/build ./build
cp -r algovivo.build.repo/build algovivo.repo/build
```

### 2. **Set up Python**
Install Python 3.11 and required dependencies:
```bash
sudo apt-get update
sudo apt-get install python3.11 python3-pip
virtualenv ma-env -p python3.11
source ma-env/bin/activate

pip install -r requirements.txt
```

### 3. **Generate trajectory**
Run the script with default or custom parameters:
```bash
export PYTHONPATH=.:algovivo.repo/utils/py
python scripts/generate_trajectory_with_attn_policy.py \
  --agent data/agents/biped \
  --steps 300 \
  --policy data/policies/attn
```
Repeat for the quadruped agent:
```bash
python scripts/generate_trajectory_with_attn_policy.py \
  --agent data/agents/quadruped \
  --steps 300 \
  --policy data/policies/attn
```

### 4. **Install FFmpeg**
```bash
sudo apt-get install ffmpeg
```

### 5. **Render frames**
```bash
cd algovivo.repo
npm ci
cd ..
node algovivo.repo/utils/trajectory/renderTrajectory.js --no-sandbox \
  --mesh trajectory_attn.out/mesh.json \
  --steps trajectory_attn.out/steps \
  --width 300 \
  --height 300
```

### 6. **Create individual videos**
```bash
ffmpeg -framerate 30 -i frames.out/%d.png -c:v libx264 -profile:v high -crf 20 -pix_fmt yuv420p video_biped.mp4
ffmpeg -framerate 30 -i frames.out/%d.png -c:v libx264 -profile:v high -crf 20 -pix_fmt yuv420p video_quadruped.mp4
```

### 7. **Merge videos side by side**
```bash
ffmpeg \
  -framerate 30 -i frames-biped/%d.png \
  -framerate 30 -i frames-quadruped/%d.png \
  -filter_complex "[0:v]pad=iw+2:color=black[left];[left][1:v]hstack=inputs=2" \
  -c:v libx264 -profile:v high -crf 20 -pix_fmt yuv420p -r 30 -y merged.mp4
```

