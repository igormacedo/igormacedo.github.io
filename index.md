# Processamento Digital de Imagens

Esta página contém os códigos desenvolvidos para a disciplinda de PDI (2018.1)

## Seção 2 - Exercícios
### Questão 1

Arquivo regions.cpp

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  Mat image;

  if (argc < 5) {
    cout << "Quatro argumentos numéricos são necessários" << endl;
    return 0;
  }

  int ax = atoi(argv[1]);
  int ay = atoi(argv[2]);
  int bx = atoi(argv[3]);
  int by = atoi(argv[4]);

  image = imread("biel.png",CV_LOAD_IMAGE_GRAYSCALE);
  if(!image.data)
    cout << "nao abriu biel.png" << endl;

  namedWindow("janela",WINDOW_AUTOSIZE);

  if (ax > 0 && ay > 0 && bx < image.rows && by < image.cols){
    for(int i=ax;i<bx;i++){
      for(int j=ay;j<by;j++){
        image.at<uchar>(i,j)= 255 - image.at<uchar>(i,j);
      }
    }
  }
  else{
    cout << "Argumentos devem ser de pontos dentro da imagem";
  }

  imshow("janela", image);  
  waitKey();

  return 0;
}

```

### Questão 2

Arquivo trocarregioes.cpp

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int, char**){
  Mat image;

  image= imread("biel.png",CV_LOAD_IMAGE_GRAYSCALE);
  if(!image.data)
    cout << "nao abriu bolhas.png" << endl;

  namedWindow("janela",WINDOW_AUTOSIZE);

  for(int i=0; i<image.rows/2; i++){
    for(int j=0; j<image.cols/2; j++){
      uchar aux = image.at<uchar>(i,j);
      image.at<uchar>(i,j)=image.at<uchar>(image.rows/2 + i, image.cols/2 + j);
      image.at<uchar>(image.rows/2 + i, image.cols/2 + j) = aux;
    }
  }

  for(int i=0; i<image.rows/2; i++){
    for(int j=image.cols/2; j<image.cols; j++){
      uchar aux = image.at<uchar>(i,j);
      image.at<uchar>(i,j)=image.at<uchar>(image.rows/2 + i, j - image.cols/2);
      image.at<uchar>(image.rows/2 + i, j - image.cols/2) = aux;
    }
  }
  
  imshow("janela", image);  
  waitKey();

  return 0;
}

```

## Seçãoo 3 - Exercícios

### Questão 1 

blabalbalablablabalablabalba

### Questão 2

Arquivo labellingHoles.cpp

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  Mat image, imgholes, imgregions, imgaux;
  int width, height;
  int nobjects;
  
  CvPoint p;
  image = imread(argv[1],CV_LOAD_IMAGE_GRAYSCALE);
  
  if(!image.data){
    std::cout << "imagem nao carregou corretamente\n";
    return(-1);
  }
  width=image.cols;
  height=image.rows;

  p.x=0;
  p.y=0;

  // Elimina elementos que tocam as bordas superior e inferior
  for(int i=0; i<height; i=i+height-1){
    for(int j=0; j<width; j++){
      if(image.at<uchar>(i,j) == 255){
        p.x=j;
        p.y=i;
        floodFill(image,p,0);
      }
	  }
  }

  // Elimina elementos que tocam as bordas laterais
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j = j+width-1){
      if(image.at<uchar>(i,j) == 255){
        p.x=j;
        p.y=i;
        floodFill(image,p,0);
      }
	  }
  }

  image.copyTo(imgholes);
  image.copyTo(imgregions);
  image.copyTo(imgaux);

  // Preenche background com fundo branco
  p.x=0;
  p.y=0;
  floodFill(imgaux,p,255);

  // HOLES DETECTION
  // detectar pontos pretos em uma imagem e preencher com identificador o pixel (anterior)
  // da outra imagem
  nobjects=255; // inicia em 255 para facilitar visualização
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(imgaux.at<uchar>(i,j) == 0){
        // achou uma região preta
        if(imgholes.at<uchar>(i,j-1) == 255){
          nobjects -= 15; // diminui 15 para facilitar visualização
          p.x=j-1;
          p.y=i;
          floodFill(imgholes,p,nobjects);
        }        
      }
	  }
  }

  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(imgholes.at<uchar>(i,j) == 255){
        p.x=j;
        p.y=i;
        floodFill(imgholes,p,0);
      }
    }
  }

  // REGIONS WITHOUT HOLES DETECTION

  nobjects=255;
  for(int i=0; i<height; i++){
    for(int j=0; j<width; j++){
      if(imgholes.at<uchar>(i,j) != 255 && imgholes.at<uchar>(i,j) != 0 && imgregions.at<uchar>(i,j) == 255){
         // diminui 15 para facilitar visualização
        p.x=j;
        p.y=i;
        floodFill(imgregions,p,0);
      }
      if(imgregions.at<uchar>(i,j) == 255){
        nobjects -= 10;
        p.x=j;
        p.y=i;
        floodFill(imgregions,p,nobjects);
      }
    }
  }

  imwrite("labelingHolesRegions.png", imgregions);
  imwrite("labelingHolesDetected.png", imgholes);
  waitKey();
  return 0;
}
```

