---
title: Developping a web-app to visualize sorting algorithms in React and Typescript
date: 2023-05-21 16:00:00 +0000
categories: [Personal Projects]
tags: [reactjs, algorithm, front-end, typescript]
---

I recently started working again on a old side project of mine, and made some good progress that I wanted to share here. Last year, I started a side project in which I wanted to display sorting algorithm as they are sorting an array. This idea is not new, and it was inspired by a youtube video. You can watch it by clicking on the following image, and learn more about the project by clicking [here](https://panthema.net/2013/sound-of-sorting/).

[![youtube thumbnail for the sound-of-sorting demo video](https://img.youtube.com/vi/kPRA0W1kECg/maxresdefault.jpg)](https://youtu.be/kPRA0W1kECg)

I was fascinated by the video the first time I watched it, and like many developper, I never actually used sorting algorithms at work, and had limited experience with them. A lot of people learn them for technical interviews, and promptly forget about them after a prolonged lack of use.

I always liked *learning by doing*, and decided that there was no better way to get experience with sorting algorithms then using and implementing them in a way that explained visually how they work. Being able to explain things usually mean that you understand them well, and if I managed to find a good way to visually explain the steps of the algorithm, then I probably understood how it worked well enough.

To that end, I started a project to allows users of the web-app to select different sorting algorithms, different array length, and different speed of sorting, allowing them to see in details how the algorithm worked. Doing so, I hoped to be able to provide a kind of "playground" that would allow people to better understand the inner workings of the different algorithms.

## V1 of the project

Back then I had never used typescript, and after a quick look at it and how it integrated with React, I decided to use javascript. I remember try to create a functional componenent and getting loads of errors, and wanting to focus on learning the algorithms I decided not to bother for now. Being a Backend developper, developping a web-app was already enough of a challenge. I elected to use Material UI component library to avoid having to battle with CSS (it still happened anyway), as it had everything I needed for this project.

### Displaying arrays of different length in the browser

The first step of the project was to able to generate random unsorted arrays.
That was quite easy:

```javascript
const [r_array, setRArray] = useState([]);
const createUnsortedArray = () => {
  for (var array=[],i=0;i<100;++i) array[i]=i;
  var tmp, current, top = array.length;
  if(top) while(--top) {
    current = Math.floor(Math.random() * (top + 1));
    tmp = array[current];
    array[current] = array[top];
    array[top] = tmp;
  }
  setRArray(array);
};
```

Then, I had to create a "canvas" on which I could display the array visually. Similarly to the sound-of-sorting project, I wanted to have colored columns of different height represent the values of the array.

I needed to do two things:

- calculate the width of the column depending on the size of the array
- calculate the height of the column depending on the value in the array

That was taken care of by the following code:

```javascript
const ArrayBit = ({ index, arrayLength }) => {
    return (
        <div
            style={{
                width: 100 / arrayLength + "%",
                height: index / arrayLength * 100 + "%",
                backgroundColor: '#282625',
                alignSelf: 'end'
            }}>
        </div>
    );
}
```

I then only needed to display the columns by iterating over the array:

```javascript
<div className="arrayHolder" > {
    r_array.length !== 0 ? (
        r_array.map((val) => {
            return <ArrayBit key={val}
                index={val}
                arrayLength={r_array.length}
            />
        })
    ) : < div className='center-loading' > Loading... </div>
}
</div>
```

At that point I was able to display the array nicely on the screen.

## Fighting against React

Well, I must say that choosing React was probably not the smartest decision. I quickly realised that React had a feature that batches updates in state to avoid doing thousands of re-render back to back. Which is exactly what I was doing.

My first implementation of a bubble sort looked like this:

```javascript
const BubbleSort = async (targetArray, updateArrayMethod, setRunningState, setStepCount) => {
    setRunningState(true)
    let arr = [...targetArray]
    let step = 0
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length; j++) {
            if (arr[j] > arr[j + 1]) {
                step++
                let temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                updateArrayMethod([...arr]);
            }
        }
    }
    setRunningState(false);
}
```

This resulted in a nice unsorted array transforming instantly into a nice sorted array. Not exactly what I was looking for.

Thankfully, there is a work around. Using a promise and the `setTimeout` function, I could force React to re-render the DOM when I wanted to. I must once again thank the folks over at StackOverflow for their help on this matter.

Here is the updated function with a delay method to force re-renders:

```javascript
const delay = () => {
    return new Promise(
        resolve => setTimeout(resolve, 0.00001)
    );
};

const BubbleSort = async (targetArray, updateArrayMethod, setRunningState, setStepCount) => {
    setRunningState(true)
    let arr = [...targetArray]
    let step = 0
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0; j < arr.length; j++) {
            await delay();
            if (arr[j] > arr[j + 1]) {
                step++
                setStepCount(step);
                let temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                updateArrayMethod([...arr]);
            }
        }
    }
    setRunningState(false);
}
```

I now had a nice web-app on which people could launch a Bubble sort and see the column move around on the canvas. Nice!

![V1 of the web-app](/assets/img/algorithm_visualizer/algo_visu_v1.png)

## Working on a V2 with Typescript

After implementing the first algorithm for the project, I stopped working on it for a while. I was unsatisfied with some aspect of code, but was busy with other responsabilities that kept me from working on it. I then simply forgot about it for about a year.

Recently I've been working with Typescript and React, and that gave me the motivation to return to this project, rewrite it, improve the code base and add more algorithm.

I ended up rewriting from scratch the project using Typescript and the latest version of React. I made several improvement and ended up with a much better code base. The web-app was also greatly improved, with the addition of colors in the bars when sorting to better see what the algorithm is actually doing.

Among the improvement, there is:

- Using Redux to manage the state of the web-app, avoiding passing props all over the place
- All the app doesn't live in the App.js file
- Better project structure
- Implemented Bubble Sort, Insertion Sort, Selection Sort and Quick Sort.
- Can now control the speed of the sorting, from very fast to veeeeeerrrryyyyy slow.
- Colored columns.
- The colums are not longer sticked to gather and now have an outline

Here is a look at the newest version:

![V2 of the web-app](/assets/img/algorithm_visualizer/algo_visu_v2.png)

There are still some improvements to be made, especially on the color management. I'm not entirely satisfied with how it's working now.

Thanks to Vercel, the web-app is available online. you can visit it on [algo.mrtnhwtt.dev](https://algo.mrtnhwtt.dev)

If you have any suggestion or notice any problem in my code, feel free to reach out! It would be greatly appreciated.

## Conclusion

Well, I still hate working with CSS. I'm sure that feeling is only born from a lack of familiarity. Other than that, I really enjoyed working on this old project of mine. It was really fun to go back on some of my old code and fix all of the dumb decisions I made. I'm also quite happy that I got to work with sorting algorithm, they are incredible. I only have respect for people that thought up those algorithm, some of them are really impressive.

I hope that you could check out my project and that it helps you understanding some of these algorithms.
