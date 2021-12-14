# Summary
This repository implements [Proximal Policy Optimization](https://medium.com/intro-to-artificial-intelligence/proximal-policy-optimization-ppo-a-policy-based-reinforcement-learning-algorithm-3cf126a7562d) for the **OpenAI CarRacing-v0** environment using an architecture based on [CoordConv layers](https://arxiv.org/abs/1807.03247).

* The **PPO** algorithm is a revision from what was shown in https://github.com/xtma/pytorch_car_caring. I use a simmilar two-headed structure for the output of the network but with a smaller image size and less complexity in the convolutional layers.
* In order to regain the accuracy lost by simplifying the model and the input size I've modified the architecture for using **CoordConv** layers instead of regular convolutional layers. My main reference was this implementation: https://github.com/walsvid/CoordConv

![image](https://user-images.githubusercontent.com/26325749/146055405-82e348bd-e11e-42f6-8cb9-bb0ae5286fd5.png)

Here you can see a run once the model is trained:

INSERT GIF HERE

# Usage

The repository is not dockerized since I experienced many issues while executing OpenAI render functions in a container due to its lack of display, as detailed here: https://stackoverflow.com/questions/40195740/how-to-run-openai-gym-render-over-a-server. The problem is that **CarRacing** environment calls the render function inside its step function since the state is the image and it has to build it, so this call is not avoidable. I've tried many proposed solutions like redirecting the display, using a wrapper or modifying the original OpenAI environment but none of them worked.

In the source folder of the project you'll find the requirements:

```
pip3 install -r requirements.txt
```

Then you can run the training script. It accepts also the --render argument if you want to check the live performance:

```
python3 src/train.py --model coordconvnet
```

Allowed models are:

* **convnet**: 6 layer Convolutional Network, as in https://github.com/xtma/pytorch_car_caring using 96x96 images as the input
* **coordconvnet**: 4 layer CoordConvolutional Network, using 48x48 images as descrived in the summary.

This generates a **model-name_date.pt** model in the **models** folder and a **model-name_date.csv** and a **model-name_date.png** in the **results/individual** folder with the training performance. This is used later for comparing the models.

Once the training is done you can execute the test script. Make sure that you use the correct model name, models are generated and stored in the **models** folder. You don't need to add the extension, only the name of the file. I provide some models in case you don't want to train your own:

```
python3 src/test.py --model-name convnet_2021-12-13--21:50:29 --render --episodes 10 --sleep-time 0.1
```

There you can evaluate the performance of the model over n episodes (default 100) and visualize the results if you choose the --render argument, with the --sleep-time argument for modifying the timestep between renders (default 0).

Finally, you can execute the visualization script. This generates model comparisons of all the models in the **models** folder over time and episode number, averaging the results with the standard deviation as the error bar:

```
python3 src/visualize.py --model-name convnet_2021-12-13--21:50:29
```

This visualizations are saved in **results/comparison** folder. You don't need to pass the **--model-name** argument, but if you do it will also use that model for some test runs, saving them as gifs with also some examples of the beta functions that choose the actions in every step, saving them in **results/samples**.

# Results

This are the comparisons between models I was talking above:

![compared_models](https://user-images.githubusercontent.com/26325749/146059217-cd30fa68-41d9-436c-ba8d-f826605bf547.png)![compared_models_time](https://user-images.githubusercontent.com/26325749/146059231-461f92f3-6fb0-4b0d-aefe-cf28a70244e1.png)

These are averaged over 5 runs for each type of model. Me can see that the performance by episode is a bit lower in the CoordConvolutional model at the beggining but after that it experiences a less erratic behavior than the baseline model. This last one is quite impredictable, in 2 of the 5 runs it started loosing performance until it almost reached an averaged reward over the last 100 episodes of zero by the end of the 2000 episodes. This volatile behavior appears also in the CoordConvolutional model but with a smaller impact, I think this is because being a smaller model makes it less prone to overfitting. 

If we check the averaged reward **by time** instead of by episode the CoordConvolutional model outperforms clearly the baseline, since it takes the half the time than the baseline for each episode.

Now we can check the behavior of the model for some particular situations. The output of the network for the action selection is the parameters of a beta distribution for evey action (turn, gas, brake):

![image](https://user-images.githubusercontent.com/26325749/146060487-87cdd779-1a7d-49f3-8845-73ad8934019f.png)![image](https://user-images.githubusercontent.com/26325749/146060615-ad22c401-4921-4454-ab88-654a7912b44b.png)![image](https://user-images.githubusercontent.com/26325749/146060686-cda619d3-0037-4b27-95c6-eb3c0a450959.png)


It is quite interesting how the brake distribution only shifts to the right slightly in the closest turns, while the rest of the time it is really narrow around a zero value.





