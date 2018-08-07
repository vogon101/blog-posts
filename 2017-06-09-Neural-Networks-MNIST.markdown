---
layout: post
title: "Deep Learning for Java (DL4J) in Scala, MNIST and custom images"
date: 2017-06-09
categories:
 - programming
 - neural-networks
 - machine-learning
 - dl4j
 - scala
---

MNIST is a data set of 10s of thousands of handwritten digits which can be used to teach a computer to recognise these numbers. It is the machine learning equivalent of "Hello world", one of the first projects that anyone learning about neural networks will attempt.

There are 1000s of tutorials on how to build a relitavely successfull classifer for this data in hundreds of different languages and tools. The thing that most of them lack, however, is showing you how to *use* your network. Most of them load the data set from a csv or by using an automated loader and the only feed back you get is a percentage of how accurate the model is.

To me this always seems unsatisfactory. I wanted to be able to write a  digit and have my network tell me what number it was. I did this using the framework [Deep Learning 4 Java](https://deeplearning4j.org/) which is a well established machine learning platform for Java. My language of choice continues to be Scala which, being JVM, works well with DL4J. To see it in action I created a simple site in the play framework which recognises digits drawn on an HTML canvas.

All the code in for this project is available [on github](https://github.com/vogon101/MachineLearning) and you can see the results [here](http://learning.vogonjeltz.com).

Step one was to set up an SBT project to work with DL4J and its numerical computing library, ND4J. I have also included a couple of dependencies that we will use later to pre-process the images.

{% highlight scala %}
libraryDependencies += "org.deeplearning4j" % "deeplearning4j-core" % "0.8.0"

libraryDependencies += "org.nd4j" % "nd4j-native-platform" % "0.8.0"

libraryDependencies ++= Seq(
  "com.twelvemonkeys.imageio" % "imageio" % "3.1.2",
  "com.twelvemonkeys.imageio" % "imageio-core" % "3.1.2",
  "com.twelvemonkeys.common" % "common-lang" % "3.1.2"
)

fork in run := true

outputStrategy in run := Some(StdoutOutput)

connectInput in run := true

javaOptions in run ++= Seq(
  "-Xms256M", "-Xmx2G", "-XX:MaxPermSize=1024M", "-XX:+UseConcMarkSweepGC")

mainClass in (Compile, run) := Some("com.vogonjeltz.machineInt.app.MNISTApplication")
{% endhighlight %}

Next I needed to create and train a model. This is not a neural network tutorial and as such I'm not going to go over this stage in detail. Instead I will point you to the [excellent tutorial](https://deeplearning4j.org/mnist-for-beginners) from DL4J's own  site. It goes over the very basics of creating a Feed-Forward MNIST classifier. With the model trained, [serialise it into ".zip" format](https://deeplearning4j.org/modelpersistence) and we have a working classifier.

Now the step that no-one tells you (not really but whatever): How to use your network.

The first step is getting images in the correct format. The MINST dataset is a set of 28x28 images that are greyscale and centred. The first step is reading in bitmap (.bmp) images. We can do this with the imageio package that's in our `build.sbt`. Next we rotate them (this is just a quirk of the formatting) through 90 degrees. The final step is to center them. The way this is done in the MNIST dataset is through "centre of mass of pixels" which basically means finding where the dark pixels are most concentrated and translating that to the centre.

In scala this looks something like the following. The function `readBWBMP` takes in the path to a black and white bmp and returns it as an array of Doubles. Using the pre-process function it also rotates and centres it.

```scala

def preProcess(image: BufferedImage, size: Int, log: Boolean = false): Array[Double] = {
    val imageArray = Array.ofDim[Double](size, size)

    val rotatedImage = new BufferedImage(size,size,image.getType())
    val g = rotatedImage.createGraphics()
    g.rotate(Math.toRadians(90), size/2,size/2)

    g.drawImage(image, 0, size, size , -size, null)

    val finalImage = new ArrayBuffer[Double]()

    var xtotal, ytotal = 0d
    var num = 0d

    for (x <- Range(0,size)) {
      for (y <- Range(0, size)) {
	val p = rotatedImage.getRGB(x, y)
        val a = 255 - ((p >> 24) & 0xff)
        val r = 255 - ((p >> 16) & 0xff)
        val g = 255 - ((p >> 8) & 0xff)
        val b = p & 0xff


        rotatedImage.setRGB(x,y,((a<<24) | (r<<16) | (g<<8) | b))
        xtotal += x * r
        ytotal += y * r
        num += r
      }
    }

    val com_1 = (xtotal/num, ytotal/num)
    if (log) println(f"Old Centre of mass: (${com_1._1}%1.2f, ${com_1._2}%1.2f)")

    val translatedImage = new BufferedImage(size,size,image.getType())
    val g_translate = translatedImage.createGraphics()

    val translate = ((14 - com_1._1) /2, (14 - com_1._2)/2)
    if (log) println(f"Translate by (${translate._1}%1.2f,${translate._2}%1.2f)")

    val tx: AffineTransform = new AffineTransform
    tx.translate(translate._1, translate._2)
    g_translate.setTransform(tx)
    g_translate.drawImage(rotatedImage, tx, null)


    xtotal = 0d
    ytotal = 0d
    num = 0d

    for (x <- Range(0,size)) {
      for (y <- Range(0, size)) {
        val colour = translatedImage.getRGB(x, y)
        val r = ((colour & 0xff0000) >> 16) / 255d
        finalImage.append(
          if (r < 0.02) 0
          else r
        )
        xtotal += x * r
        ytotal += y * r
        num += r
      }
    }

    val com_2 = (Math.round(xtotal/num), Math.round(ytotal/num))
    if (log) println(f"New Centre of mass: (${com_2._1}, ${com_2._2}%1.2f)\n")

    finalImage.toArray
}


def readBWBMP(path: String, size: Int, log: Boolean = false): Array[Double] = {
    if (log) println(s"Loading image $path")
    val image = ImageIO.read(new File(path))
    preProcess(image, size,log)
}
```

With the image loaded we can use our model to recognise it. This is actually not that easy to find out how to do this but the actual method itself is really simple. Bellow we contvert our pixel array into an INDArray, the internal format used by DL4J and pass it into the model.

```scala
def use(pathToImage: String) : (Int, Array[Double])= {
    val imageData = Utils.readBWBMP(pathToImage, 28)
    val input = new NDArray(imageData.map(_.toFloat))
    use(input)
}

def use(input: INDArray, log: Boolean): (Int, Array[Double]) = {
    val output = model.output(input).dup.data.asDouble
    var bestGuess = (-1d, -1)
    for ((x, i) <- output.zipWithIndex) {
      if (log) println(f"$i - $x%1.2f")
      if (x > bestGuess._1) bestGuess = (x, i)
    }
    (bestGuess._2, output)
}
```

And that's it. We now have a method that can be called on an image path (`use(path)`). The member `model` is an unserialised version of the model.

Now for the website ([learning.vogonjeltz.com/MNIST](http://learning.vogonjeltz.com/MNIST)). This takes in a couple of elements. A playframework based API which takes an image json-formatted as a post parameter and gives that to the model. The next stage is to build the page that has a canvas on it, allow the user to draw on it, serialise it and json-encode it and finally send it via AJAX to the server. Thats a lot to go over in code snippets so the full project is [here](https://github.com/vogon101/MachineLearning/tree/master/WebMachineLearning).


