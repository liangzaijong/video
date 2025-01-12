#  Video Diffusion Models

## Updates
**[2025.1]** The LLM receives text prompts to generate dynamic scene layouts, including reasoning statements, captioned bounding boxes for objects in each frame, and background prompts, with templates available in prompt.py. The Doubao model ep-20250112204250-tmph4 is used to generate layouts, and benchmark comparisons are conducted with GPT-3.5 and GPT-4. To enhance video smoothness, OpenCV is employed for frame interpolation optimization, increasing the frame rate from 24 fps to 36 fps through motion compensation interpolation, significantly improving video fluidity.

## Installation
Install the dependencies:
```
pip install -r requirements.txt
```

## Stage 1: Generating Dynamic Scene Layouts (DSLs) from Text

Note: We have uploaded the layout caches for the benchmark onto this repository, so you can skip this step if you don't need to generate layouts for new prompts (i.e., if you only want to use LLM-generated layouts for benchmarking in stage 2).

If you want to re-generate layouts for the same prompts, you need to remove the cache in the cache directory.

Our Layout Generation Format: The LLM takes in a text prompt describing the image and outputs dynamic scene layouts consisting of three elements: 1. a reasoning statement for analysis, 2. a captioned box for each object in each frame, and 3. a background prompt. The template and example prompts are in prompt.py. You can modify the template, example prompts, and parsing function to ask the LLM to generate additional elements or even perform chain-of-thought reasoning for better generation.

Automated Query from Doubao API
If you just want to evaluate stage 2 (layout-to-video stage), you can skip stage 1 as we have already uploaded the layout caches to this repository. You don't need a Doubao API key for stage 2.

If you have access to the Doubao API, you can set the API key in utils/api_key.py or set the DOUBAO_API_KEY environment variable. Then, you can use the Doubao API for batch text-to-layout generation by querying the LLM, with the model ep-20250112204250-tmph4 as an example:

``` shell
复制
python prompt_batch.py --prompt-type demo --model ep-20250112204250-tmph4 --auto-query --always-save --template_version v0.1
--prompt-type demo includes a few prompts for demonstration. You can modify them in prompt.py. The layout generation will be cached to avoid querying the LLM again with the same prompt, which reduces costs.
```
You can visualize the dynamic scene layouts in the form of bounding boxes in img_generations/imgs_demo_templatev0.1. They are saved as gif files. For horizontal video generation with zeroscope, the square layout will be scaled according to the video aspect ratio.

Run Our Benchmark on Text-to-Layout Generation Evaluation
We provide a benchmark that applies to both stage 1 and stage 2. This benchmark includes a set of prompts with five tasks (numeracy, attribution, visibility, dynamic spatial, and sequential) as well as unified benchmarking code for all implemented methods and both stages.

This will generate layouts from the prompts in the benchmark (with --prompt-type lvd) and evaluate the results:

``` shell
复制
python prompt_batch.py --prompt-type lvd --model ep-20250112204250-tmph4 --auto-query --always-save --template_version v0.1
python scripts/eval_stage_one.py --prompt-type lvd --model ep-20250112204250-tmph4 --template_version v0.1

Dataset Collection and Comparison
We collected datasets using three different models for comparison:
```


## Stage 2: Video Optimization
Frame Rate Improvement
During the video generation process, the original model's output frame rate might be low, resulting in less smooth videos. To improve the frame rate, we use OpenCV to perform frame interpolation on the generated video, thereby increasing the frame rate.

Motion Compensation Interpolation Method
Assumption: Objects move at a constant linear velocity.
The method directly maps the motion trajectory of blocks from reference frame t-1 (or t+1) to reference frame t+1 (or t-1) and interpolates the frames in between.

Experimental Results
Original Frame Rate: 24 fps.

Optimized Frame Rate: 36 fps.

Smoothness Improvement: By using frame interpolation, the video's frame rate increased by 1.5 times, significantly improving the smoothness of the video.

## References

Lian, Long, Baifeng Shi, Adam Yala, Trevor Darrell, and Boyi Li. 2023. "LLM-grounded Video Diffusion Models." n The Twelfth International Conference on Learning Representations (ICLR).

Lian, Long, Boyi Li, Adam Yala, and Trevor Darrell. 2023. "LLM-grounded Diffusion: Enhancing Prompt Understanding of Text-to-Image Diffusion Models with Large Language Models." arXiv preprint arXiv:2305.13655. 
