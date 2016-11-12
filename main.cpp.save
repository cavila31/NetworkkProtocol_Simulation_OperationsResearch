#include <iostream>
#include <random>
#include <list>
#include <vector>
#include <limits.h>
#include <stdlib.h>
#include <time.h>
#include <ctime>
#include <chrono>
#include <math.h>
#include <thread>
#include <algorithm>


using namespace std;

struct Msg
{
    int num_msg;
    int hora_entrada;
    int veces_enviado;
    bool eviado = false;
    int codigo;
};

struct ACK
{
    int num_msg_esperando;
    int veces_enviado;
    bool se_pierde=false;
};

struct find_num_msg : std::unary_function<Msg, bool>
{
    int num_msg;
    find_num_msg(int num_msg):num_msg(num_msg) { }
    bool operator()(Msg const& m) const
    {
        return m.num_msg == num_msg;
    }
};

double ultimo_cambio_colaA=0;
double Eventos[6]= {INT_MAX, INT_MAX, INT_MAX, INT_MAX, INT_MAX, INT_MAX};
unsigned int colaA_tamanno=0;
unsigned int colaB=0;
unsigned int colaAckA = 0;
unsigned int  primerosA[20] = {};
unsigned int primerosB [20] = {};
unsigned int indiceColaA = 0;
double reloj = 0;
double Max = 0;
unsigned int ventana = 0;
unsigned int ventanaEnviada = 0;
bool libreA = true;
bool libreB = true;
vector<Msg> msgA;
list<Msg> msgB;
vector<Msg> ventanaA;
list <Msg> ultimo;
list<Msg> ultimo2;
list<ACK> ackToA;
list<double>ackReloj;
ACK ultimoAck;
unsigned int ultimoAckRecibido = 1;
unsigned int ultimoAckEnviado = 1;
unsigned int ultimoMsgEnviadoA = 0;
double DuracionTimer=0;
double TimersVentana[8]= {};
unsigned int punteroTimers=0;
unsigned int cantMsgsGenerados=0;
double MsgsCorrectos=0;
double ZTPCA=0;
double ZTP_SIST=0;
double ZPTTM=0;
unsigned int frameCorrectosB = 0;

double revisarFrame_ACK ()
{


    srand( unsigned(time(NULL)));

    double r=((double)rand()/(double)RAND_MAX);

    // cout<<" R= "<<r<<endl;
    double x=sqrt(5*r + 4);
    //    cout<<" X= "<<x<<endl;

    return x;
}

double normalDistr(double media , double varianza)
{
    srand( unsigned(time(NULL)));

    double r1=((double)rand()/(double)RAND_MAX);
    double r2=((double)rand()/(double)RAND_MAX);

    //cout<<" R1= "<<r1<<"\tR2= "<<r2<<endl;
    double z=0;
    if((2*3.14159265*r2)!=0)
    {

        z= sqrt((-2*(log(r1))))*sin(2*3.14159265*r2);
    }
    else
    {
        z= sqrt((-2*(log(r1))))*cos(2*3.14159265*r2);
    }
    //cout<<" z= "<<z<<endl;

    double x=(varianza*z)+media;
    //cout<<" X= "<<x<<endl;

    return x;
}

double expDistr(double lambda)
{
    srand( unsigned(time(NULL)));

    double r=((double)rand()/(double)RAND_MAX);
//   cout<<" R= "<<r<<endl;


    return (-(log(r) )/lambda);
}

int Next_Event ()
{

    int smallest = Eventos[0];
    int index=0;
    for (int i = 0; i < 6; i++)
    {
        cout<<"Evento[";
        cout<<i;
        cout<<"]: ";
        cout<<Eventos[i]<<endl;

        if (Eventos[i] < smallest)
        {

            smallest = Eventos[i];
            index=i;
        }
    }
    return index;

}

Msg GeneradorMsg ()
{
    Msg m;
    cantMsgsGenerados++;
    m.num_msg=cantMsgsGenerados;
    m.veces_enviado=0;
    return m;
}
int Fin()
{
    int retorno = -1;

    if (reloj>=Max)
    {

        //ESTADISTICAS

        //Tamaño promedio de la cola en A
        double TPCA= ZTPCA/reloj;
        //Tiempo promedio de "permanencia de un mensaje en el sistema"

        double TP_SIST= ZTP_SIST/MsgsCorrectos;

        //Promedio del tiempo de transmisión para un mensaje
        double PTTM=ZPTTM/MsgsCorrectos;
        // cout<<"ZPTTM= "<<ZPTTM<<endl;
        // cout<<"MsgsCorrectos= "<<MsgsCorrectos<<endl;

        //Tiempo promedio de servicio

        double TPS=TP_SIST-PTTM;

        //Eficiencia
        double Eficiencia= PTTM/TPS;

        string estadisticas= "\n\t*************************************************************\n"
                             "\t*                      Estadisticas                         *\n"
                             "\t*                                                           *\n";

        // "\t\t* Eficiencia:                                 *\n"
        string fin= "\t*                                                           *\n"
                    "\t*************************************************************\n";
        cout<<estadisticas<<"\t* Tamanno promedio de la cola en A: "<<TPCA<<"                 *\n"<< "\t* Tiempo prom de permanencia de un mgs en el sist: "<<TP_SIST<<"  *\n"<<"\t* Prom del tiempo de transmision para un msg: "<< PTTM<<"          *\n"<<"\t* Tiempo promedio de servicio: "<<TPS<< "               *\n"<< "\t* Eficiencia: "<< Eficiencia<<"                                 *\n"<<fin<<endl;
    }
    else
    {

        int next = Next_Event();
        retorno = next;

    }
    return retorno;
}
int LlegaMsgA()
{
    Msg msg = GeneradorMsg();
    reloj= Eventos[0];
    // cout<<"reloj "<<reloj<<endl;
    double a=colaA_tamanno;
    ZTPCA+= a*(reloj-ultimo_cambio_colaA);
    colaA_tamanno++;
    ultimo_cambio_colaA=reloj;
    msg.hora_entrada=reloj;
    msgA.push_back(msg);
    if (ventana < 8)
    {
        primerosA[ventana] = msg.num_msg;
        cout<<"Entro en A: ";
        cout<<primerosA[ventana]<<endl;
        cout<<"ventana "<<ventana<<endl;
        ventanaA.push_back(msg);
        if (libreA)
        {
            libreA = false;
            Msg *msg2 = &ventanaA[ventanaEnviada];
            //ventanaA.pop_front();
            double frame = expDistr(0.5);
            reloj += frame;
            //generar si se va a perder el frame, llega bien o con error
            srand(time(0));
            int cod = rand() % 100 + 1;
            //  cout<<"cod "<<cod<<endl;

            if( cod<=10)
            {
                msg2->codigo=3;
            }
            else if (cod>15)
            {
                msg2->codigo=1;
            }
            else
            {
                msg2->codigo = 2;
            }
            //  cout<<"msg.codigo "<<msg.codigo<<endl;

            //bien: msg.codigo=1;
            //error: msg.codigo=2;
            //perdido: msg.codigo=3;

            //  libreA = true;
            msg2->eviado = true;
            Eventos[1]=reloj+1;
            //ultimo.push_back(msg);
            ventanaEnviada++;
            ultimo2.push_back(*msg2);
        }
        ventana++;
    }
    else
    {
        bool fin = false;
        int contador = 8;
        while (!fin)
        {
            if (contador > 19)
                fin = true;
            else
            {
                if (primerosA[contador] != 0)
                    contador++;
                else
                    fin = true;

            }
        }
        if (contador > 19)
            primerosA[contador] = msg.num_msg;
    }

    double sgtMsg = normalDistr(25, 1);
    reloj += sgtMsg;
    //cout<<reloj
    Eventos[0]=reloj;
    //  cout<<"Eventos[0] "<<Eventos[0]<<endl;

    return Fin();
}

int SeLiberaA (Msg *msg)
{

    cout<<msg->num_msg<<endl;
    indiceColaA++;
    reloj= Eventos[1];
    ultimoMsgEnviadoA = msg->num_msg;

    msg->veces_enviado+=1;
    ZPTTM++;

    if (msg->codigo!=3)
    {
        Eventos[2]=reloj+1;
        ultimo.push_back(*msg);
    }


    if (Eventos[5]==INT_MAX)
    {
        TimersVentana[0]=reloj+DuracionTimer;
        punteroTimers=1;
        Eventos[5]=reloj+DuracionTimer;
    }
    else
    {
        TimersVentana[punteroTimers]=reloj+DuracionTimer;
        punteroTimers++;
    }
    if (ventanaA.size() <= ventanaEnviada)
    {
        libreA=true;
        Eventos[1] = INT_MAX;
    }
    else if (ventanaEnviada < 8)
    {
        cout<<"Tamano ventana: ";
        cout<<ventanaA.size()<<endl;
        cout<<"Ventana enviada: ";
        cout<<ventanaEnviada<<endl;
        msg = &ventanaA[ventanaEnviada];
        libreA=false;
        double frame = expDistr(0.5);
        reloj += frame;
        //generar si se va a perder el frame, llega bien o con error
        srand(time(0));
        int cod = rand() % 100 + 1;
        // cout<<"cod "<<cod<<endl;

        if( cod<=10)
        {
            msg->codigo=3;
        }
        else if (cod>15)
        {
            msg->codigo=1;
        }
        else
        {
            msg->codigo = 2;
        }
        // cout<<"msg.codigo "<<msg.codigo<<endl;

        //bien: msg.codigo=1;
        //error: msg.codigo=2;
        //perdido: msg.codigo=3;

        //  libreA = true;
        msg->eviado = true;
        Eventos[1]=reloj+1;
        //  cout<<"Eventos[1] "<<Eventos[1]<<endl;
        //colaA--;

        //Eventos[1]=reloj+1;
        //cout<<"Eventos[1] "<<Eventos[1]<<endl;
        ventanaEnviada++;
        ultimo2.push_back(*msg);
    }
    else
    {
        libreA=true;
        Eventos[1] = INT_MAX;
    }
    ultimo2.pop_front();
    return Fin();


}

int LlegaFrameB (Msg *msg)
{

    reloj=Eventos[2];
    if(msg->codigo!=3)
    {
        colaB++;
        msgB.push_back(*msg);
        if (msg->num_msg != ultimoAck.num_msg_esperando)
        {
            msg->codigo = 2;
            cout<<"No era"<<endl;
        }

        if (libreB == true)
        {
            libreB = false;
            int t_rev = revisarFrame_ACK();
            ACK a;
            if (msg->codigo == 2)
                a.num_msg_esperando = ultimoAckEnviado;
            else
            {
                a.num_msg_esperando=msg->num_msg + 1;
                frameCorrectosB++;
                for (unsigned int contador = 19; contador > 0; contador--)
                    primerosB[contador] = primerosB[contador-1];
                primerosB[0] = msg->num_msg;
            }
            ultimoAckEnviado = a.num_msg_esperando;

            //generar si ACK se pierde

            srand(time(0));
            int perdido = rand() % 100 + 1;

            if (perdido<=15)
            {
                a.se_pierde = true;
            }

            if( a.se_pierde==false )
            {
                if (Eventos[4] < INT_MAX)
                    ackReloj.push_back(reloj+t_rev+.25+1);
                else
                    Eventos[4]=reloj+t_rev+.25+1;
                ackToA.push_back(a);
                colaAckA++;
            }
            Eventos[3]=reloj+t_rev+.25;
            ultimoAck = a;

            //return Fin();
        }
    }
    ultimo.pop_front();
    Eventos[2] = INT_MAX;
    return Fin();
}

int SeLiberaB(ACK ack)
{
    reloj= Eventos[3];
    msgB.pop_front();
    if (colaB > 0)
        colaB--;
    // cout<<"reloj "<<reloj<<endl;
    libreB = true;
    if (colaB > 0)
    {
        cout<<"Hay cola en b: ";
        cout<<colaB<<endl;
        Msg *msg = &msgB.front();
        if (libreB==true)
        {
            libreB = false;
            int t_rev= revisarFrame_ACK();
            ACK a;
            if (msg->codigo == 2)
                a.num_msg_esperando=ultimoAckEnviado;
            else
            {
                a.num_msg_esperando=msg->num_msg + 1;
                frameCorrectosB++;
                for (unsigned int contador = 19; contador > 0; contador--)
                    primerosB[contador] = primerosB[contador-1];
                primerosB[0] = msg->num_msg;
            }
            ultimoAckEnviado = a.num_msg_esperando;
            //generar si ACK se pierde

            srand(time(0));
            int perdido = rand() % 100 + 1;

            if (perdido<=15)
            {
                a.se_pierde = true;
            }

            if( a.se_pierde==false )
            {
                if (Eventos[4] < INT_MAX)
                    ackReloj.push_back(reloj+t_rev+.25+1);
                else
                    Eventos[4]=reloj+t_rev+.25+1;
                ackToA.push_back(a);
                colaAckA++;
            }
            Eventos[3]=reloj+t_rev+.25;
            ultimoAck = a;
            ultimoAckEnviado = a.num_msg_esperando;

        }
    }
    else
        Eventos[3] = INT_MAX;

    return Fin();
}

int ARecibeACK(ACK ack)
{
    cout<<ackToA.size()<<endl;
    ackToA.pop_front();
    colaAckA--;
    reloj = Eventos[4];
    //int i = ventanaA.front().num_msg;
    int i = ack.num_msg_esperando;

    //  cout<<"I: "<<i<<endl;
    ZTPCA+=(colaA_tamanno*(reloj-ultimo_cambio_colaA));
    MsgsCorrectos += (i - ultimoAckRecibido);
    colaA_tamanno -= (i - ultimoAckRecibido);   //despues de un largo analisis no tengo idea de para q puso eso, pero modifica la cola!
    ultimoAckRecibido = ack.num_msg_esperando;
    ultimo_cambio_colaA=reloj;


    std::vector<Msg>::iterator it = std::find_if (msgA.begin(), msgA.end(), find_num_msg(ack.num_msg_esperando-1));
    ZTP_SIST+=reloj-(*it).hora_entrada;
    //  cout<<"duracion msj "<<reloj-(*it).hora_entrada<<endl;
    ventanaA.clear();
    ventana = 0;
    ventanaEnviada = 0;
    Eventos[5] = INT_MAX;
    for (int i = 0; i <20 ; i ++)
    {
        if (primerosA[i] != 0)
            primerosA[i] = 0;
    }
    bool fin = false;
    while (!fin)
    {

        if (ack.num_msg_esperando + ventana > msgA.back().num_msg)
            fin = true;
        else
        {
            msgA[ack.num_msg_esperando + ventana-1].eviado = false;
            ventanaA[ventana] = msgA[ack.num_msg_esperando + ventana-1];
            cout<<msgA[ack.num_msg_esperando + ventana-1].num_msg<<endl;
            cout<<ack.num_msg_esperando + ventana<<endl;
            primerosA[ventana] = ventanaA[ventana].num_msg;
            ventana++;
            if (ventana > 7)
                fin = true;
        }
    }
    if (ventana > 0)
    {
        ventana = 0;
        Msg *msg = &ventanaA[ventana];
        indiceColaA = (msg->num_msg-1);

        if (libreA)
        {

            libreA=false;
            double frame = expDistr(0.5); //xq ud usa .5, si es de 2 segundos?
            reloj += frame;
            //generar si se va a perder el frame, llega bien o con error
            srand(time(0));
            int cod = rand() % 100 + 1;
            //      cout<<"cod "<<cod<<endl;

            if( cod<=10)
            {
                msg->codigo=3;
            }
            else if (cod>15)
            {
                msg->codigo=1;
            }
            else
            {
                msg->codigo = 2;
            }
            //    cout<<"msg.codigo "<<msg.codigo<<endl;

            //bien: msg.codigo=1;
            //error: msg.codigo=2;
            //perdido: msg.codigo=3;

            //  libreA = true;
            msg->eviado = true;
            Eventos[1]=reloj+1;
            //   cout<<"Eventos[1] "<<Eventos[1]<<endl;
            //colaA--;
            //ultimo.push_back(*msg);
            msg->veces_enviado+=1;
            //      cout<<"2El msj: "<< msg.num_msg<<" enviado= "<<msg.veces_enviado<<endl;
            ZPTTM++;
            ultimo2.push_back(*msg);
        }
        //Eventos[1]=reloj+1;
        //cout<<"Eventos[1] "<<Eventos[1]<<endl;

    }
    if (colaAckA > 0)
    {
        ultimoAck = ackToA.front();
        Eventos[4] = ackReloj.front();
        ackReloj.pop_front();
    }
    else
        Eventos[4] = INT_MAX;
    return Fin();
}

int SeVenceTimer(Msg *msg)
{
    reloj=Eventos[5];
    //ultimo.num_msg=msg->num_msg;

    Eventos[5] = INT_MAX;
    ventana = ventanaA.size();
    ventanaEnviada = 0;
    if (ventana > 0)
    {

        if (libreA)
        {
            msg = &ventanaA[ventanaEnviada];
            cout<<msg->num_msg<<endl;
            indiceColaA = (msg->num_msg-1);
            libreA=false;
            double frame = expDistr(0.5);
            reloj += frame;
            //generar si se va a perder el frame, llega bien o con error
            srand(time(0));
            int cod = rand() % 100 + 1;
            // cout<<"cod "<<cod<<endl;

            if( cod<=10)
            {
                msg->codigo=3;
            }
            else if (cod>15)
            {
                msg->codigo=1;
            }
            else
            {
                msg->codigo = 2;
            }
            //  cout<<"msg.codigo "<<msg.codigo<<endl;

            //bien: msg.codigo=1;
            //error: msg.codigo=2;
            //perdido: msg.codigo=3;

            //  libreA = true;
            msg->eviado = true;
            Eventos[1]=reloj+1;
            //  cout<<"Eventos[1] "<<Eventos[1]<<endl;
            //colaA--;
            //ultimo.push_back(*msg);
            msg->veces_enviado+=1;
            //  cout<<"3El msj: "<< msg.num_msg<<" enviado= "<<msg.veces_enviado<<endl;
            ZPTTM++;
            //Eventos[1]=reloj+1;
            //cout<<"Eventos[1] "<<Eventos[1]<<endl;
            ventanaEnviada++;
            ultimo2.push_back(*msg);
        }

    }


    return Fin();
}

int main()
{
    colaB = 0;
    Eventos[0]=1;
    int fin = 0;
    bool lento=false;
    int l=0;
    ultimoAck.num_msg_esperando = 1;
    string creditos =       "\n\t\t\t************************************\n"
                            "\t\t\t*  Proyecto de Simulacion 1        *\n"
                            "\t\t\t*                                  *\n"
                            "\t\t\t*  Carolina Avila Arias    B30721  *\n"
                            "\t\t\t*  Daniel Peralta Madriz   B04740  *\n"
                            "\t\t\t************************************\n\n";

    cout<<creditos<<endl;
    cout<<"\n\t\t Digite el Tiempo de Simulacion"<<endl;
    cin >> Max;

    //cin.ignore();
    cout<<"\t\t Digite la Duracion del timer"<<endl;
    cin >> DuracionTimer;
    //cin.ignore();

    string modo="\t\t Desea ver la simulacion correr en modo lento?\n"
                "\t\t   Digite 1 Si \n"
                "\t\t   Digite 2 No  \n";
    cout<<modo<<endl;
    cin >> l;
    //cin.ignore();

    if (l==1)
    {

        lento=true;
        //cout<<"si, lento"<<endl;

    }
    string lentoString="\t\t Cuando este listo(a) para continuar, presione cualquier número\n";
    //cout<<l<<endl;
    //Sleep(10);

    while (fin >= 0)
    {
        switch(fin)
        {
        case 0:
        {
            fin = LlegaMsgA();
        }
        break;
        case 1:
        {
            fin = SeLiberaA(&ultimo2.front());
        }
        break;
        case 2:
        {
            fin =  LlegaFrameB(&ultimo.front());
        }
        break;
        case 3:
        {
            fin = SeLiberaB(ultimoAck);
        }
        break;
        case 4:
        {
            fin = ARecibeACK(ultimoAck);
        }
        break;
        case 5:
        {
            fin = SeVenceTimer(&ultimo.back());
        }
        break;
        }
        cout<<"Reloj: ";
        cout<<reloj<<endl;
        cout<<"Tamano de la cola: ";
        cout<<colaA_tamanno<<endl;
        for(int i = 0; i < 20 ; i++)
        {
            if (primerosA[i] == 0)
                i = 20;
            else
            {
                cout<<"Primer[";
                cout<<i;
                cout<<"] en A es: ";
                cout<<primerosA[i]<<endl;
            }
        }
        cout<<"Ultimo msg enviado: ";
        cout<<ultimoMsgEnviadoA<<endl;
        cout<<"Ultimo Ack recibido por A es: ";
        cout<<ultimoAckRecibido<<endl;
        cout<<"Ultimo ack enviado por B: ";
        cout<<ultimoAckEnviado<<endl;
        cout<<"Total de frames recibidos corrector por B: ";
        cout<<frameCorrectosB<<endl;


        for(int i = 0; i < 20 ; i++)
        {
            if (primerosB[i] == 0)
                i = 20;
            else
            {
                cout<<"Primer[";
                cout<<i;
                cout<<"] en B es: ";
                cout<<primerosB[i]<<endl;
            }
        }

        if (fin >= 0)
            cout<<"Evento siguiente: "<<fin<<endl;

        cout<<"\n\n"<<endl;
        if (lento && fin >= 0)
        {
            cout<<lentoString<<endl;
            cin >> l;
        }


    }


    //cout<<"fin "<<fin<<endl;



    return 0;
}
