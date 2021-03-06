# Processamento Digital de Imagens

Esta página contém os códigos desenvolvidos para a disciplinda de PDI (2018.1)

## Seção 2 - Exercícios
### Questão 1

Este programa tem a função de selecionar uma determinada região e trocar os tons de cinza de forma que a região selecionada será o inverso da imagem real.
Arquivo regions.cpp

![Regions](regions.png)

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

Esse programa tem a simples função de trocar os quadrantes da imagem, gerando uma imagem embaralhada de acordo com a imagem de input.

Arquivo trocarregioes.cpp

![trocar regiões](trocarregioes.png)

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

Caso existem mais de 255 objetos para identificar na imagem, ocorrerá erro pois a forma de identificação que estamos utilizado é de colorir em tons de cinza que vão de 1 a 254 os objetos encontrados. Dessa forma é possível reconhecer individualmente cada objeto. Porém, se quisermos identificar mais de 255 objetos, podemos utlizar uma representação em ponto flutuante, para assim termos um número muito maior de possibilidades. Nesse caso, será dificil de visualizar a imagem gerada, mas o computador poderá identificar os objetos encontrados.

### Questão 2

Esse programa foi desenvolvido para identificar regiões com e sem buracos. O resultado final é que, dado uma imagem de input, ele irá criar uma imagem que contém todas as regiões sem buracos numeradas de forma única (usando a coloração em tons de cinza) e uma outra imagem que contém todas as regiões com buraco (um ou mais buracos) numeradas em coloração de tons de cinza também.

Arquivo labellingHoles.cpp

![Original](bolhas.png)
![Buracos detectados](labelingHolesDetected.png)
![Regiões detectadas](labelingHolesRegions.png)

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
## Seçãoo 3 - Exercícios

### Questão 1

Este program tem a função de capturar frames do video da webcam e gerar duas imagens.  A primeira imagem é o frame original e a segunda imagem é o frame com histograma equalizado pela função do OpenCV. A gente percebe que o histograma equalizado permite a melhor visualização de cores, especialmente em ambientes muito claros ou muito escuros onde o histograma original tinha valores muito concentrados em uma única região.

Arquivo equalize.cpp

![Equilize](equalize.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  Mat image, grey_image, eq_image, dst_image;
  int width, height;
  VideoCapture cap;
  vector<Mat> planes;
  Mat histR, histG; //, histB;
  int nbins = 64;
  float range[] = {0, 256};
  const float *histrange = { range };
  bool uniform = true;
  bool acummulate = false;

  cap.open(0);
  
  if(!cap.isOpened()){
    cout << "cameras indisponiveis";
    return -1;
  }
  
  width  = cap.get(CV_CAP_PROP_FRAME_WIDTH);
  height = cap.get(CV_CAP_PROP_FRAME_HEIGHT);

  cout << "largura = " << width << endl;
  cout << "altura  = " << height << endl;

  int histw = nbins, histh = nbins/2;
  Mat histImgR(histh, histw, CV_8UC1, Scalar(0)); // modificado para criar um histograma em greyscale
  Mat histImgG(histh, histw, CV_8UC1, Scalar(0));

  //Mat histImgR(histh, histw, CV_8UC3, Scalar(0,0,0));
  //Mat histImgG(histh, histw, CV_8UC3, Scalar(0,0,0));
  //Mat histImgB(histh, histw, CV_8UC3, Scalar(0,0,0));

  while(1){
    cap >> image;

    cvtColor(image, grey_image, cv::COLOR_BGR2GRAY);
    // Apply Histogram Equalization
    equalizeHist(grey_image, eq_image);
    
    //split(image, planes);

    calcHist(&grey_image, 1, 0, Mat(), histR, 1,
             &nbins, &histrange,
             uniform, acummulate);
    calcHist(&eq_image, 1, 0, Mat(), histG, 1,
             &nbins, &histrange,
             uniform, acummulate);

    // calcHist(&planes[0], 1, 0, Mat(), histR, 1,
    //          &nbins, &histrange,
    //          uniform, acummulate);
    // calcHist(&planes[1], 1, 0, Mat(), histG, 1,
    //          &nbins, &histrange,
    //          uniform, acummulate);
    // calcHist(&planes[2], 1, 0, Mat(), histB, 1,
    //          &nbins, &histrange,
    //          uniform, acummulate);

    normalize(histR, histR, 0, histImgR.rows, NORM_MINMAX, -1, Mat());
    normalize(histG, histG, 0, histImgG.rows, NORM_MINMAX, -1, Mat());
    // normalize(histB, histB, 0, histImgB.rows, NORM_MINMAX, -1, Mat());

    histImgR.setTo(Scalar(0));
    histImgG.setTo(Scalar(0));
    // histImgB.setTo(Scalar(0));
    
    for(int i=0; i<nbins; i++){
      line(histImgR,
           Point(i, histh),
           Point(i, histh-cvRound(histR.at<float>(i))),
           Scalar(255), 1, 8, 0);
      line(histImgG,
           Point(i, histh),
           Point(i, histh-cvRound(histG.at<float>(i))),
           Scalar(255), 1, 8, 0);
      // line(histImgB,
      //      Point(i, histh),
      //      Point(i, histh-cvRound(histB.at<float>(i))),
      //      Scalar(255, 0, 0), 1, 8, 0);
    }
    histImgR.copyTo(grey_image(Rect(0, 0 ,nbins, histh)));
    histImgG.copyTo(eq_image(Rect(0, 0 ,nbins, histh)));
    // histImgB.copyTo(image(Rect(0, 2*histh ,nbins, histh)));

    hconcat(grey_image, eq_image, dst_image);

    imshow("image", dst_image);
    if(waitKey(30) >= 0) break;
  }
  return 0;
}
```

### Questão 2

Arquivo motiondetection.cpp

Este programa de motion detection utiliza o histograma como ferramenta para reconhecer movimentos na tela. Isso é possível pois o histograma deve sofre uma variação notável em sua correlação com cada frame anterior ao movimento. Assim é possivel identificar e sinalizar o movimento quando acontece.

```cpp
#include <iostream>
#include <stdlib.h>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char** argv){
  Mat image, grey_image, eq_image, dst_image;
  int width, height;
  VideoCapture cap;
  vector<Mat> planes;
  Mat histR, old_hist; //, histB;
  int nbins = 64;
  float range[] = {0, 256};
  const float *histrange = { range };
  bool uniform = true;
  bool acummulate = false;

  cap.open(0);
  
  if(!cap.isOpened()){
    cout << "cameras indisponiveis";
    return -1;
  }
  
  width  = cap.get(CV_CAP_PROP_FRAME_WIDTH);
  height = cap.get(CV_CAP_PROP_FRAME_HEIGHT);

  cout << "largura = " << width << endl;
  cout << "altura  = " << height << endl;

  int histw = nbins, histh = nbins/2;
  Mat histImgR(histh, histw, CV_8UC1, Scalar(0)); // modificado para criar um histograma em greyscale

  // // faz uma vez para poder, calcular a primeira correlação, dar continuidade dentro do loop
  cap >> image;
  cvtColor(image, grey_image, cv::COLOR_BGR2GRAY);
  calcHist(&grey_image, 1, 0, Mat(), histR, 1,
             &nbins, &histrange,
             uniform, acummulate);
  normalize(histR, histR, 0, histImgR.rows, NORM_MINMAX, -1, Mat());
  
  histR.copyTo(old_hist);

  while(1){
    cap >> image;

    cvtColor(image, grey_image, cv::COLOR_BGR2GRAY);

    calcHist(&grey_image, 1, 0, Mat(), histR, 1,
             &nbins, &histrange,
             uniform, acummulate);

    normalize(histR, histR, 0, histImgR.rows, NORM_MINMAX, -1, Mat());

    float sum = 0, sqrtold = 0, sqrtnew = 0;

    double d = compareHist(histR,old_hist,CV_COMP_CORREL);
    cout << d << endl;

    if ( d < 0.99)
    {
      cout << "Moviemnto detectado";
    }

    histR.copyTo(old_hist);

    histImgR.setTo(Scalar(0));
    
    for(int i=0; i<nbins; i++){
      line(histImgR,
           Point(i, histh),
           Point(i, histh-cvRound(histR.at<float>(i))),
           Scalar(255), 1, 8, 0);
    }
    histImgR.copyTo(grey_image(Rect(0, 0 ,nbins, histh)));

    imshow("image", grey_image);
    if(waitKey(30) >= 0) break;
  }
  return 0;
}

```

## Seção 5 - Exercícios

### Questão 1

Este programa implementa uma nova função ao programa do exemplo explicado nessa seção do tutorial. Agora é implementado o laplaciano do gaussino e por isso foi preciso recalcular a matriz kernel do filtro e alterar o programa para que ele fosse implementado com essa nova função. Vemos que o laplaciano do gaussiano permite uma melhor identficação das bordas.

![Comparação](lapgau.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

void printmask(Mat &m){
  for(int i=0; i<m.size().height; i++){
    for(int j=0; j<m.size().width; j++){
      cout << m.at<float>(i,j) << ",";
    }
    cout << endl;
  }
}

void menu(){
  cout << "\npressione a tecla para ativar o filtro: \n"
	"a - calcular modulo\n"
    "m - media\n"
    "g - gauss\n"
    "v - vertical\n"
	  "h - horizontal\n"
    "l - laplaciano\n"
    "y - laplaciano do gaussiano\n"
	  "esc - sair\n";
}

int main(int argvc, char** argv){
  VideoCapture video;
  float media[] = {1,1,1,
				   1,1,1,
				   1,1,1};
  float gauss[] = {1,2,1,
				   2,4,2,
				   1,2,1};
  float horizontal[]={-1,0,1,
					  -2,0,2,
					  -1,0,1};
  float vertical[]={-1,-2,-1,
					0,0,0,
					1,2,1};
  float laplacian[]={0,-1,0,
					 -1,4,-1,
					 0,-1,0};
  float laplaciangauss[]={0,0,1,0,0,
  					0,1,2,1,0,
  					1,2,-16,2,1,
  					0,1,2,1,0,
  					0,0,1,0,0};

  Mat cap, frame, frame32f, frameFiltered;
  Mat mask(3,3,CV_32F), mask1;
  Mat result, result1;
  double width, height, min, max;
  int absolut;
  char key;
  
  video.open(0); 
  if(!video.isOpened()) 
    return -1;
  width=video.get(CV_CAP_PROP_FRAME_WIDTH);
  height=video.get(CV_CAP_PROP_FRAME_HEIGHT);
  std::cout << "largura=" << width << "\n";;
  std::cout << "altura =" << height<< "\n";;

  namedWindow("filtroespacial",1);

  mask = Mat(3, 3, CV_32F, media); 
  scaleAdd(mask, 1/9.0, Mat::zeros(3,3,CV_32F), mask1);
  swap(mask, mask1);
  absolut=1; // calcs abs of the image

  menu();
  for(;;){
    video >> cap; 
    cvtColor(cap, frame, CV_BGR2GRAY);
    flip(frame, frame, 1);
    imshow("original", frame);
    frame.convertTo(frame32f, CV_32F);
    filter2D(frame32f, frameFiltered, frame32f.depth(), mask, Point(1,1), 0);
    if(absolut){
      frameFiltered=abs(frameFiltered);
    }
    frameFiltered.convertTo(result, CV_8U);
    imshow("filtroespacial", result);
    key = (char) waitKey(10);
    if( key == 27 ) break; // esc pressed!
    switch(key){
    case 'a':
	  menu();
      absolut=!absolut;
      break;
    case 'm':
	  menu();
      mask = Mat(3, 3, CV_32F, media);
      scaleAdd(mask, 1/9.0, Mat::zeros(3,3,CV_32F), mask1);
      mask = mask1;
      printmask(mask);
      break;
    case 'g':
	  menu();
      mask = Mat(3, 3, CV_32F, gauss);
      scaleAdd(mask, 1/16.0, Mat::zeros(3,3,CV_32F), mask1);
      mask = mask1;
      printmask(mask);
      break;
    case 'h':
	  menu();
      mask = Mat(3, 3, CV_32F, horizontal);
      printmask(mask);
      break;
    case 'v':
	  menu();
      mask = Mat(3, 3, CV_32F, vertical);
      printmask(mask);
      break;
    case 'l':
	  menu();
      mask = Mat(3, 3, CV_32F, laplacian);
      printmask(mask);
      break;
    case 'y':
	  menu();
      mask = Mat(5, 5, CV_32F, laplaciangauss);
      printmask(mask);
      break;
    default:
      break;
    }
  }
  return 0;
}


```

## Seção 6 - Exercícios

### Questão 1

Este programa implementa um filto Tilt Shift que permite deixar apenas parte da imagem em foco e o resto defocado. Ele utiliza um filtro de média 3x3 e faz uma junção da imagem filtrada com a imagem original. É possível controlar a altura, a posição e o decaimento da parte original a partir de sliders. Para iniciar o programa é preciso chamar como `./tiltshift filename`

![Comparação](tiltshift.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <math.h>

using namespace cv;
using namespace std;

double altura;
int altura_slider = 0;
int altura_slider_max = 100;

double decaimento;
int decaimento_slider = 0;
int decaimento_slider_max = 100;

double posicao;
int posicao_slider = 0;
int posicao_slider_max = 100;

Mat image, image32f, imagefiltered32f, imagefiltered;
Mat blended;
Mat imageTop;

Mat mask(3, 3, CV_32F), maskMedia;

char TrackbarName[50];

double alfa(int i, double a, double d, double p)
{
    return 0.5 * (tanh((i - p) / d) - tanh((i - a - p) / d));
}

void on_trackbar_altura(int, void *)
{
    for (int i = 0; i < image.rows; i++)
    {
        double a = alfa(i, altura_slider, decaimento_slider+1, posicao_slider);
        addWeighted(image.row(i), a, imagefiltered.row(i), 1 - a, 0, blended.row(i));
    }
    imshow("tiltshift", blended);
}

void on_trackbar_decaimento(int, void *)
{
    on_trackbar_altura(altura_slider, 0);
}

void on_trackbar_posicao(int, void *)
{
    on_trackbar_altura(altura_slider, 0);
}

int main(int argvc, char **argv)
{
    image = imread(argv[1]);
    blended = Mat::zeros(image.rows, image.cols, CV_8UC3);
    altura_slider_max = image.rows;
    posicao_slider_max = image.rows;

    float media[] = {1, 1, 1,
                     1, 1, 1,
                     1, 1, 1};

    mask = Mat(3, 3, CV_32F, media);
    scaleAdd(mask, 1 / 9.0, Mat::zeros(3, 3, CV_32F), maskMedia);

    image.convertTo(image32f, CV_32F);
    filter2D(image32f, imagefiltered32f, image32f.depth(), maskMedia, Point(1, 1), 0);
    imagefiltered32f = abs(imagefiltered32f);
    imagefiltered32f.convertTo(imagefiltered, CV_8U);

    namedWindow("tiltshift", 1);

    sprintf(TrackbarName, "Altura (0-%d)", altura_slider_max);
    createTrackbar(TrackbarName, "tiltshift",
                   &altura_slider,
                   altura_slider_max,
                   on_trackbar_altura);
    on_trackbar_altura(altura_slider, 0);

    sprintf(TrackbarName, "Decaimento (0-%d)", decaimento_slider_max);
    createTrackbar(TrackbarName, "tiltshift",
                   &decaimento_slider,
                   decaimento_slider_max,
                   on_trackbar_decaimento);
    on_trackbar_decaimento(decaimento_slider, 0);

    sprintf(TrackbarName, "Posição (0-%d)", posicao_slider_max);
    createTrackbar(TrackbarName, "tiltshift",
                   &posicao_slider,
                   posicao_slider_max,
                   on_trackbar_posicao);
    on_trackbar_posicao(posicao_slider, 0);

    waitKey(0);

    imwrite( "blended.png", blended );
    return 0;
}

```

### Questão 2

Esse programa adiciona o efeito de Tilt Shift em um arquivo e depois disso salva o vídeo com um efeito de stop motion, pois utiliza apenas alguns frames da imagem. Executar ele indicando o nome do arquivo `./stopmotion filename`

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <math.h>

using namespace cv;
using namespace std;

Mat image, image32f, imagefiltered32f, imagefiltered;
Mat blended;
Mat imageTop;

Mat mask(3, 3, CV_32F), maskMedia;

char TrackbarName[50];

double alfa(int i, double a, double d, double p)
{
    return 0.5 * (tanh((i - p) / d) - tanh((i - a - p) / d));
}

int main(int argvc, char **argv)
{

    VideoCapture cap (argv[1]);

    if (!cap.isOpened()){
        cout << "Failed to open input file " << argv[1] << endl;
        exit(1);
    }

    VideoWriter out ("stopmotion.avi", CV_FOURCC('D','I','V','X'), 
                     cap.get(CV_CAP_PROP_FPS)/5,
                     Size(cap.get(CV_CAP_PROP_FRAME_WIDTH), 
                     cap.get(CV_CAP_PROP_FRAME_HEIGHT)));


    while(1)
    {
        cap >> image;
        if(image.empty()) exit(0);

        //image = imread(argv[1]);
        blended = Mat::zeros(image.rows, image.cols, CV_8UC3);

        float media[] = {1, 1, 1,
                        1, 1, 1,
                        1, 1, 1};

        mask = Mat(3, 3, CV_32F, media);
        scaleAdd(mask, 1 / 9.0, Mat::zeros(3, 3, CV_32F), maskMedia);

        image.convertTo(image32f, CV_32F);
        filter2D(image32f, imagefiltered32f, image32f.depth(), maskMedia, Point(1, 1), 0);
        imagefiltered32f = abs(imagefiltered32f);
        imagefiltered32f.convertTo(imagefiltered, CV_8U);

        for (int i = 0; i < image.rows; i++)
        {
            double a = alfa(i, image.rows*0.15, 1, image.rows/2);
            addWeighted(image.row(i), a, imagefiltered.row(i), 1 - a, 0, blended.row(i));
        }

        out << blended;

    }

    return 0;
}


```

## Seção 7 - Exercícios

### Questão 1

Este programa implemente o filtro homomórfico com a opção de selecionar valores para as variáveis de interesse no filtro. Este filtro deve permitir bloquear certas frequencias em uma imagem de pouca luminosidade, possívelmente revelando detalhes que são pouco visíveis na imagem original.

![Original](originalfiltro.png)
![Com Filtro](filtro.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <opencv2/imgproc/imgproc.hpp>

#define RADIUS 20

using namespace cv;
using namespace std;

int gl_slider = 0;
int gl_slider_max = 100;

int gh_slider = 50;
int gh_slider_max = 100;

int c_slider = 5;
int c_slider_max = 100;

int d0_slider = 5;
int d0_slider_max = 200;

Mat imageOriginal;
Mat padded;
Mat_<float> realInput, zeros;
Mat complexImage;
Mat filter, tmp;
vector<Mat> planos;

int dft_M, dft_N;

// troca os quadrantes da imagem da DFT
void deslocaDFT(Mat& image){
  Mat tmp, A, B, C, D;

  // se a imagem tiver tamanho impar, recorta a regiao para
  // evitar cópias de tamanho desigual
  image = image(Rect(0, 0, image.cols & -2, image.rows & -2));
  int cx = image.cols/2;
  int cy = image.rows/2;
  
  // reorganiza os quadrantes da transformada
  // A B   ->  D C
  // C D       B A
  A = image(Rect(0, 0, cx, cy));
  B = image(Rect(cx, 0, cx, cy));
  C = image(Rect(0, cy, cx, cy));
  D = image(Rect(cx, cy, cx, cy));

  // A <-> D
  A.copyTo(tmp);  D.copyTo(A);  tmp.copyTo(D);

  // C <-> B
  C.copyTo(tmp);  B.copyTo(C);  tmp.copyTo(B);
}

cv::Mat create_homomorfic_filter(cv::Size paddedSize, double gl, double gh, double c, double d0){
  Mat filter = Mat(paddedSize, CV_32FC2, Scalar(0));
  Mat tmp = Mat(dft_M, dft_N, CV_32F);

  for(int i=0; i < tmp.rows; i++){
    for(int j=0; j < tmp.cols; j++){
        float coef = (i-dft_M/2)*(i-dft_M/2)+(j-dft_N/2)*(j-dft_N/2);
        tmp.at<float> (i,j) = (gh-gl)*(1.0 - (float)exp(-(c*coef/(d0*d0)))) + gl;
    }
  }

  // cria a matriz com as componentes do filtro e junta
  // ambas em uma matriz multicanal complexa  
  Mat comps[]= {tmp, tmp};
  merge(comps, 2, filter);
  return filter;
}

void on_trackbar_move(int, void*){
  // limpa o array de matrizes que vao compor a
  // imagem complexa
  planos.clear();

  // cria a compoente real e imaginaria (zeros)
  realInput = Mat_<float>(padded);
  //realInput += Scalar::all(1);
  //log(realInput,realInput);

  // insere as duas componentes no array de matrizes
  planos.push_back(realInput);
  planos.push_back(zeros);

  // combina o array de matrizes em uma unica
  // componente complexa
  // prepara a matriz complexa para ser preenchida
  complexImage = Mat(padded.size(), CV_32FC2, Scalar(0));
  merge(planos, complexImage);

  // calcula o dft
  dft(complexImage, complexImage);

  // realiza a troca de quadrantes
  deslocaDFT(complexImage);
  resize(complexImage,complexImage,padded.size());
  normalize(complexImage,complexImage,0,1,CV_MINMAX);

  // aplica o filtro frequencial
  float gl = (float) gl_slider / 100.0;
  float gh = (float) gh_slider / 100.0;
  float d0 = 25.0 * d0_slider / 100.0;
  float c  = (float)  c_slider  / 100.0;

  cout << "gl = " << gl << endl;
  cout << "gh = " << gh << endl;
  cout << "d0 = " << d0 << endl;
  cout << "c = "  << c  << endl;

  Mat filter = create_homomorfic_filter(padded.size(), gl, gh, c, d0);
  mulSpectrums(complexImage,filter,complexImage,0);

  // troca novamente os quadrantes
  deslocaDFT(complexImage);

  // calcula a DFT inversa
  idft(complexImage, complexImage);

  // limpa o array de planos
  planos.clear();

  // separa as partes real e imaginaria da
  // imagem filtrada
  split(complexImage, planos);

  // // normaliza a parte real para exibicao
  normalize(planos[0], planos[0], 0, 1, CV_MINMAX);
  imshow("Homomorphic Filter", planos[0]);
}

int main(int argc, char** argv){

  imageOriginal = imread(argv[1],CV_LOAD_IMAGE_GRAYSCALE);
  imshow("Original", imageOriginal);

  // identifica os tamanhos otimos para
  // calculo do FFT
  dft_M = getOptimalDFTSize(imageOriginal.rows);
  dft_N = getOptimalDFTSize(imageOriginal.cols);

  // realiza o padding da imagem
  copyMakeBorder(imageOriginal, padded, 0,
                 dft_M - imageOriginal.rows, 0,
                 dft_N - imageOriginal.cols,
                 BORDER_CONSTANT, Scalar::all(0));

  zeros = Mat_<float>::zeros(padded.size());

  char TrackbarName[50];

  namedWindow("Homomorphic Filter", WINDOW_NORMAL);

  sprintf( TrackbarName, "Gamma L x %d", gl_slider_max );
  createTrackbar( TrackbarName, "Homomorphic Filter", &gl_slider, gl_slider_max, on_trackbar_move);

  sprintf( TrackbarName, "Gamma H x %d", gh_slider_max );
  createTrackbar( TrackbarName, "Homomorphic Filter", &gh_slider, gh_slider_max, on_trackbar_move);

  sprintf( TrackbarName, "C x %d", c_slider_max);
  createTrackbar( TrackbarName, "Homomorphic Filter", &c_slider, c_slider_max, on_trackbar_move);

  sprintf( TrackbarName, "Cutoff Frequency x %d", d0_slider_max );
  createTrackbar( TrackbarName, "Homomorphic Filter", &d0_slider, d0_slider_max, on_trackbar_move);

  on_trackbar_move(0, NULL);

  waitKey(0);
  return 0;
}

```

## Seção 8 - Exercícios

### Questão 1

Este programa implementa uma técnica de pontilhismo utilizando o algoritmo de reconhecimento de bordas de Canny. Esta técnica gera uma imagem pontilhada de uma figura (utilizando tamanhos de raio aleatórios e deslocados aleatoriamente) e utiliza canny para reforçar as bordas e melhorar o entendimento dos contornos da figura. Para as bordas detectadas por canny, o algoritmo cria circulos de mesma cor do píxel, mas de tamanhos aleatórios, porém sem o deslocamento

![Pontilhismo](cannypontos.png)

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>
#include <fstream>
#include <iomanip>
#include <vector>
#include <algorithm>
#include <numeric>
#include <ctime>
#include <cstdlib>

using namespace cv;
using namespace std;

#define STEP 5
#define JITTER 3

int top_slider = 10;
int top_slider_max = 200;

int pointradi_slider = 3;
int pointradi_slider_max = 10;

int pointradicanny_slider = 3;
int pointradicanny_slider_max = 10;

char TrackbarName[50];

Mat image_gray, image_color;
Mat border, frame, points;

int width, height, gray;
int x, y;

vector<int> yrange;
vector<int> xrange;

void on_trackbar_canny(int, void *)
{
  Canny(image_gray, border, top_slider, 3 * top_slider);

  iota(xrange.begin(), xrange.end(), 0);
  iota(yrange.begin(), yrange.end(), 0);

  for (uint i = 0; i < xrange.size(); i++)
  {
    xrange[i] = xrange[i] * STEP + STEP / 2;
  }

  for (uint i = 0; i < yrange.size(); i++)
  {
    yrange[i] = yrange[i] * STEP + STEP / 2;
  }

  points = Mat(height, width, CV_8UC3, Scalar(255, 255, 255));

  for (auto i : xrange)
  {
    random_shuffle(yrange.begin(), yrange.end());
    for (auto j : yrange)
    {
      x = i + rand() % (2 * JITTER) - JITTER + 1;
      y = j + rand() % (2 * JITTER) - JITTER + 1;
      Vec3b color = image_color.at<Vec3b>(x, y);
      circle(points, cv::Point(y, x), pointradi_slider + rand() % 4, Scalar(color), -1, CV_AA);
    }
  }

  for (auto i : xrange)
  {
    random_shuffle(yrange.begin(), yrange.end());
    for (auto j : yrange)
    {
      if (border.at<uchar>(i,j) == 255)
      {
        Vec3b color = image_color.at<Vec3b>(i, j);
        circle(points, cv::Point(j, i), pointradicanny_slider + rand() % 2, Scalar(color), -1, CV_AA);
      }
    }
  }

  imshow("CannyPontos", points);
}

int main(int argc, char **argv)
{

  srand(time(0));
  //image = imread(argv[1],CV_LOAD_IMAGE_GRAYSCALE);
  image_color = imread(argv[1]);

  if (!image_color.data)
  {
    cout << "nao abriu" << argv[1] << endl;
    cout << argv[0] << " imagem.jpg";
    exit(0);
  }

  cvtColor(image_color, image_gray, CV_RGB2GRAY);

  width = image_color.size().width;
  height = image_color.size().height;

  xrange.resize(height / STEP);
  yrange.resize(width / STEP);

  sprintf(TrackbarName, "Threshold inferior %d", top_slider_max);

  namedWindow("CannyPontos", 1);
  createTrackbar(TrackbarName, "CannyPontos",
                 &top_slider,
                 top_slider_max,
                 on_trackbar_canny);
  createTrackbar("Point Radius", "CannyPontos",
                 &pointradi_slider,
                 pointradi_slider_max,
                 on_trackbar_canny);
  createTrackbar("Canny point radius", "CannyPontos",
                 &pointradicanny_slider,
                 pointradicanny_slider_max,
                 on_trackbar_canny);

  on_trackbar_canny(top_slider, 0);

  waitKey();
  return 0;
}

```

## Seção 9 - Exercícios

### Questão 1

Este programa implemente o filtro homomórfico com a opção de selecionar valores para as variáveis de interesse no filtro. A imagem abaixo mostra as 10 rodadas realizadas com o algoritmo de kmeans para segmentação de cores. Este algoritmo deve escolher centros que irão definir quais as cores mais importantes da imagem e por fim, a imagem será recriada utilizando apenas um conjunto bem reduzido de cores. Se caso os centros iniciais para o algoritmo kmeans forem gerados de maneira aleatória, o resultado final para cada centro deverá ser diferente. Por isso notamos que a imagem varia entre uma rodada e outra, pois os centros estão sendo gerados de maneira aleatória e o resultado final da segmentação de cores vai diferir a cada rodada do algorítmo. A diferença, mesmo sendo pouca, é suficiente para ser notada visualmente na maioria dos casos.

![Kmeans](colorgif.gif)
