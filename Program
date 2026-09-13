#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <math.h>
#include <stdbool.h>
#define HLEN 8192


struct nodo_stazione
{
  int el;
  struct nodo_stazione *next; //il prossimo nel bucket
  struct nodo_stazione *succ; //il successivo nella lista
  struct nodo_stazione *prev; //il precedente nella lista
  int Maxheap_car[513]; //ne metto 513 in questo modo lo rappresento come un maxheap ossia con la cella 0 mai occupata
  int last; // mi indica l'ultimo elemento che ho inserito
};

struct nodo_percorso 
{
  int el;
  struct nodo_percorso *p; //parent
  struct nodo_percorso *prev; //nodo che ho messo in precedenza per poi far le free. 
};


struct nodo_viste
{
    int el;
    struct nodo_viste *next; 
};

struct nodo_queue
{
    int el;
    struct nodo_queue *next;
    struct nodo_percorso *figlio_di; //sin dalla  coda devo sapere sove attaccare il nodo che devo attacccare. 
};

struct queue
{
    struct nodo_queue *head;
    struct nodo_queue *tail;
};

struct nodo_res
{
  int el;
  struct nodo_res *next;
};


typedef struct queue queue; 
typedef struct nodo_stazione *list; // list sarebbero le liste nella tabella hash ossia dei puntatori a nodo stazione
typedef struct nodo_percorso *pp; //puntatore percorso. 
typedef struct nodo_res *list_res; 
typedef struct nodo_viste *sv; //stazioni viste

list aggiungi_stazione(int KMstazione, list *hash_stazioni, list A); 
list cerca_stazione(int KMstazione, list *hash_stazioni);
list demolisci_stazione(int KMstazione,list *hash_stazioni, list A);
list sistema_puntatori(list A, list t);//presa la lista a e il puntatore al nodo che devo rimuovere prepara i puntatori per la rimozione 
void aggiungi_auto(list stazione, int KMauto); 
int cerca_auto(list stazione, int KMauto, int subT);
void rottama_auto(list stazione, int pos);
void pianifica_percorso_LtoR(int kmS1, int kmS2, list *hash_stazioni);
void pianifica_percorso_RtoL(int kmS2, int kmS1, list *hash_stazioni);//scambio le stazioni rispetto a prima così ho sempre la stazione a km minore prima
void enqueue(queue *q, int el, pp padre); 
void dequeue(queue *q); //attenzione: la dequeue toglie solo il primo nodo senza restituire nulla. 
void printhash(list l);
void printlist(list l);
list_res inserisci_testa(list_res l, int el);
void print_delete(list_res res);
void clean_hash(sv *stazioni_viste); 
void printqueue(queue q);
bool check(sv *stazioni_viste, int key); //controlla che non abbia visto la stazione che devo incodare, sia nella coda che nelle già viste
void clean_queue(queue q);
void clean_list(pp pred); 
sv vista_stazione(sv *stazioni_viste, int key);
void clean_stazioni(list *stazioni);
pp aggiungi_percorso(int key, pp figlio_di, pp prev);
int cerca_autonomiaMAX(list S1, list S2);


int main()
{
    char instruction[20];
    int kmauto, kmstazione, pos; //pos indica la posizione dell'auto da rottamare
    int s1, s2; //le 2 stazioni che fanno da pèartenza e da arrivo in pianifica_percorso
    int i;
    char c;
    bool right=1; 
    list stazioni[HLEN]; 
    list a=NULL; //lista AUTOSTRADA che poi userò per fare le mie ricerche
    list l=NULL; 
    for(i=0;i<HLEN;i++) //non so se necessario ma nel dubbio lo inizializzo con NULL
    {
        stazioni[i]=NULL; 
    }
    while(1)
    {
        i=0;
        c='z';
        while(c!=' ')
        {
            c=getc_unlocked(stdin); 
            instruction[i]=c;
            i++;
        }
        instruction[i-1]='\0';
        if(strcmp(instruction,"aggiungi-stazione") == 0)
        {
            right=right&&scanf("%d",&kmstazione);
            a=aggiungi_stazione(kmstazione, stazioni, a);
        }
        
        else if(strcmp(instruction,"demolisci-stazione") == 0)
        {
            right=right&&scanf("%d",&kmstazione);
            a=demolisci_stazione(kmstazione,stazioni,a); 
        }
        
        else if(strcmp(instruction,"aggiungi-auto") == 0)
        {
            right=right&&scanf("%d",&kmstazione);
            right=right&&scanf("%d",&kmauto);
            l=cerca_stazione(kmstazione,stazioni); 
            if(l==NULL)
                printf("non aggiunta\n");
            else 
            {
                aggiungi_auto(l,kmauto); 
                printf("aggiunta\n");
            }
            
        }
        
        else if(strcmp(instruction,"rottama-auto") == 0)
        {
            right=right&&scanf("%d",&kmstazione);
            right=right&&scanf("%d",&kmauto);
            l=cerca_stazione(kmstazione,stazioni); 
            if(l==NULL)
                printf("non rottamata\n");
            else 
            {
                pos=cerca_auto(l,kmauto,1); 
                if(pos==-1)
                {
                    printf("non rottamata\n");
                }
                else
                {
                    rottama_auto(l,pos); 
                    printf("rottamata\n");
                }
            }
        }
        else if(strcmp(instruction,"pianifica-percorso") == 0)
        {
            //printf("\nLISTA STAZIONI\n");
            //printlist(a);
            right=right&&scanf("%d",&s1);
            right=right&&scanf("%d",&s2);
            if(s1<=s2)
                pianifica_percorso_LtoR(s1,s2,stazioni);
            else 
                pianifica_percorso_RtoL(s1,s2,stazioni);
        }
        c=getc_unlocked(stdin); 
        /*if(c!='\n'&&c!='\0')
        {
            printf("ERRORE, ho letto %c\n",c);
        }*/
        if(c==EOF)
        {
        	return 0;
        }
    }
    //clean_stazioni(stazioni);
    return 0;
}

list aggiungi_stazione(int KMstazione, list *hash_stazioni, list A) 
{
    int hash;
    list temp=NULL;
    int Nauto, kmauto, i;
    hash=KMstazione%(HLEN);
    list cur=hash_stazioni[hash];
    bool right=1; 
    //aggiungo stazione alla hash table controllando che non sia un duplicato
    if(hash_stazioni[hash]==NULL)
    {
        temp=calloc(1,sizeof(struct nodo_stazione)); 
        temp->el=KMstazione;
        temp->last=0; 
        temp->next=NULL;
        hash_stazioni[hash]=temp; 
    }
    else
    {
        if(cur->el==KMstazione)
        {
            printf("non aggiunta\n");
            right=right&&scanf("%d",&Nauto); 
            for(i=1;i<=Nauto;i++)
            {
                right=right&&scanf("%d",&kmauto); 
            }
            if(right==0)
            {
                printf("errore\n");
            }
            return A; 
        }
        while(cur->next!=NULL)
        {
            if(cur->next->el==KMstazione)
            {
                printf("non aggiunta\n");
                right=right&&scanf("%d",&Nauto); 
                for(i=1;i<=Nauto;i++)
                {
                    right=right&&scanf("%d",&kmauto); 
                }
                if(right==0)
                {
                    printf("errore\n");
                }
            }
            cur=cur->next; 
        }
        temp=calloc(1,sizeof(struct nodo_stazione)); 
        temp->el=KMstazione; 
        temp->last=0; 
        temp->next=NULL; 
        cur->next=temp; 
    }
    
    
    //aggiungo stazione alla linked list nella giusta posizione tanto so già che non è un duplicato
    cur=A; //stavolta uso cur per muovermi nella lista
    list prec=NULL; 
    if(A==NULL)
    {
        A=temp;
        A->prev=NULL;
        A->next=NULL; 
    }
    else if((A->el)>(temp->el))
    {
        temp->succ=A;
        A->prev=temp; 
        temp->prev=NULL;
        A=temp;
    }
    else 
    {
        while(cur != NULL && temp->el > cur->el)
        {
            prec=cur;
            cur=cur->succ; 
            if(cur==NULL)//se nel frattempo però dovessi raggiugere l'elemento finale mi blocco
                break; 
        }
        if(cur!=NULL)
            cur->prev=temp; //se non ho raggiunto la fine sistemo il puntatore del successivo
        temp->succ=cur; 
        prec->succ=temp;
        temp->prev=prec;
    }
    
    right=right&&scanf("%d",&Nauto); 
    for(i=1;i<=Nauto;i++)
    {
        right=right&&scanf("%d",&kmauto); 
        aggiungi_auto(temp,kmauto);
    }
    printf("aggiunta\n");
    if(right==0)
    {
        printf("errore\n");
    }
    return A;
}

list cerca_stazione(int KMstazione, list *hash_stazioni) 
{
    int hash;
    hash=KMstazione%(HLEN);
    list cur=hash_stazioni[hash]; 
    if(cur==NULL)
        return NULL; 
    else 
    {
        if(cur->el==KMstazione)
        {
            return cur; 
        }
        while(cur->next!=NULL)
        {
            if(cur->next->el==KMstazione)
            {
                return cur->next; 
            }
            cur=cur->next; 
        }
    }
    return NULL; 
}


list demolisci_stazione(int KMstazione, list *hash_stazioni, list A) 
{
   int hash;
    hash=KMstazione%(HLEN);
    list prev=NULL;
    list cur=hash_stazioni[hash]; 
    if(cur==NULL)
    {
        printf("non demolita\n");
        return A; 
    }
    else if(cur->el==KMstazione)
    {
        hash_stazioni[hash]=cur->next;
        A=sistema_puntatori(A,cur);
        free(cur);
        printf("demolita\n");
        return A;
    }
    else 
    {
        while(cur->next!=NULL)
        {
            prev=cur; 
            cur=cur->next; 
            if(cur->el==KMstazione)
            {
                prev->next=cur->next;
                A=sistema_puntatori(A,cur);
                free(cur);
                printf("demolita\n");
                return A; 
            }
        }
    }
    printf("non demolita\n");
    return A; 
}

list sistema_puntatori(list A, list t)//restituisce il nuovo A in caso ho rimosso dalla cima
{
    if(A->el==t->el) //se el è il primo elemento della lista A
        {
            if(A->succ!=NULL)
                (A->succ)->prev=NULL;
            A=A->succ;
        }
        else 
        {
            if(t->succ!=NULL)
                t->succ->prev=t->prev; 
            t->prev->succ=t->succ;
        }
    return A; 
}



void aggiungi_auto(list stazione, int KMauto)
{
    (stazione->last)++; 
    int pos=stazione->last; 
    int temp;
    int p;//parent
    stazione->Maxheap_car[pos]=KMauto; 
    while(1)
    {
        p=floor(pos/2);//il genitore in un maxheap.
        if(p<=0)
            break; 
        else if((stazione->Maxheap_car[pos])>(stazione->Maxheap_car[p]))
        {
            temp=stazione->Maxheap_car[p]; 
            stazione->Maxheap_car[p]=stazione->Maxheap_car[pos]; 
            stazione->Maxheap_car[pos]=temp;
            pos=p; 
        }
        else 
        {
            break; 
        }
    }
    //codice che se necessario mi stampa il minheap 
    /*for(int i=0;i<=stazione->last;i++)
            {
                printf("%d   ",stazione->Maxheap_car[i]);
            }
            printf("\n");*/ 
   
    return;
}

int cerca_auto(list stazione, int KMauto, int subT) //il valore subT mi serve per tenere traccia del sottoalbero che sto guardando
{
    int pos=-1; //ritorna la posizione nell'array dell'auto o -1 se non l'ha trovata
    if(stazione->Maxheap_car[subT]==KMauto)
    {
        return subT;
    }
    if((2*subT<=(stazione->last))&&((stazione->Maxheap_car[2*subT])>=KMauto)&&(pos==-1))
    {
        pos=cerca_auto(stazione, KMauto, 2*subT);
    }
    if ((2*subT+1<=(stazione->last))&&((stazione->Maxheap_car[2*subT+1])>=KMauto)&&(pos==-1))
    {
        pos=cerca_auto(stazione, KMauto, 2*subT+1);
    }
    return pos;
}

void rottama_auto(list stazione, int pos)
{
    int max, temp;
    stazione->Maxheap_car[pos]=stazione->Maxheap_car[(stazione->last)]; 
    (stazione->last)--;
    
    /*printf("\n\n passaggio intermedio \n\n");
    for(int i=1;i<=stazione->last;i++)
    {
        printf("(%d)%d ",i,stazione->Maxheap_car[i]);
    }
    printf("\n\n");*/
    
    //percolate up
    while(1)
    {
        if(pos<=1||stazione->Maxheap_car[pos]<stazione->Maxheap_car[(int)floor(pos/2)])
            break; 
        else
        {
            temp=stazione->Maxheap_car[pos]; 
            stazione->Maxheap_car[pos]=stazione->Maxheap_car[pos/2]; 
            stazione->Maxheap_car[pos/2]=temp; 
            pos=pos/2;
        }
    }
    //percolate down
    while(1)
    {
        max=pos;
        if(2*pos>(stazione->last))
        {
            return; //ritorno quando sono arrivato ad una foglia ossia quando 2*pos>stazione->last
        }
        if((stazione->Maxheap_car[pos])<(stazione->Maxheap_car[2*pos]))
        {
            max=2*pos;//avendo già controllato di non essere in una foglia controllo se il figlio sx è maggiore di me, se si lo imposto a max
        }
        if((2*pos+1<=(stazione->last))&&(stazione->Maxheap_car[max]<stazione->Maxheap_car[2*pos+1]))
        {
            max=2*pos+1;
        }
        if(pos==max)
            return;
        else
        {
            temp=stazione->Maxheap_car[pos];
            stazione->Maxheap_car[pos]=stazione->Maxheap_car[max];
            stazione->Maxheap_car[max]=temp;
            /*printf("\nho scambiato l'elemento %d con l'elemento %d.\nNUOVO MAXHEAP:\n",stazione->Maxheap_car[pos],stazione->Maxheap_car[max]);
            for(int i=1;i<=stazione->last;i++)
            {
                printf("(%d)%d ",i,stazione->Maxheap_car[i]);
            }
            printf("\n");*/
            pos=max;
        }
    }
    
}


void pianifica_percorso_LtoR(int kmS1, int kmS2, list *hash_stazioni)
{
    list S1=NULL,S2=NULL; 
    S1=cerca_stazione(kmS1,hash_stazioni);
    S2=cerca_stazione(kmS2,hash_stazioni);
    list cur=S1, vic=NULL;//vicini
    pp padre=NULL;//mi serve per ricordarmi chi era il padre di chi
    int autonomia=0;
    queue q;
    bool found=0;
    q.head=NULL; 
    q.tail=NULL;
    int maxstazione=0;
    pp temp=NULL; 
    pp pred=NULL;
    list_res res=NULL;
    if(S1==NULL||S2==NULL)
    {
        printf("MI AVEVI DETTO LE STAZIONI C'ERANO!\n");
        return;
    }
    if(kmS1==kmS2)
    {
        printf("%d\n",kmS1);
        return; 
    }
    pred=aggiungi_percorso(S1->el,NULL,NULL);  
    padre=pred; //il padre e il predecessore ora sono uguali al nodo da cui parto ossia S1->el
    vic=cur->succ;
    do
    {
        if(cur->last>=1)// se c'è almeno una macchina nella stazione prendo la macchina ad autonomia maggiore. 
            autonomia=cur->Maxheap_car[1];
        else //senno setto autonomia a 0. 
            autonomia=0;
        do 
        {
            
            if((autonomia>=vic->el-cur->el) && (vic->el>maxstazione))
            {
                if(vic==S2)
                {
                    enqueue(&q, vic->el, padre);
                    found=1;
                    break; 
                }
                enqueue(&q, vic->el, padre);
                maxstazione=vic->el;
            }
            else
                break;
            vic=vic->succ;
        }while(1);
        
        if((q.head)==NULL)//se in qualsiasi momento non mi rimane nella coda "da guardare nessuno nodo ho finito
        {
            printf("nessun percorso\n");
            clean_list(pred);
            return; 
        }
        if(found==0)
        {
            padre=aggiungi_percorso(q.head->el, q.head->figlio_di, pred); //quando andrò a fare le enque le assegno col valore padre
            pred=padre; //poi ogniqualvolta aggiungo un nodo setto anche il valore predeccesor che mi serve a capire l'ultimo nodo da me inserito.
            cur=cerca_stazione(q.head->el,hash_stazioni);
            vic=cerca_stazione(maxstazione,hash_stazioni);
            vic=vic->succ; 
            //printf("sto per fare la dequeue del valore %d\n",q.head->el);
            dequeue(&q); 
        }
        else
        {
            padre=aggiungi_percorso(q.tail->el, q.tail->figlio_di, pred);
            pred=padre;
            break;
        }
    } while(1);
    temp=pred; //imposto temp all'ultimo nodo che ho messo ossia quello finale.
    while(temp!=NULL)
    {
        res=inserisci_testa(res, temp->el);
        temp=temp->p;
    }
    print_delete(res); 
    clean_list(pred);
    clean_queue(q);
    return;
}

void pianifica_percorso_RtoL(int kmS2, int kmS1, list *hash_stazioni)
{
    list S1=NULL,S2=NULL; 
    S1=cerca_stazione(kmS1,hash_stazioni);
    S2=cerca_stazione(kmS2,hash_stazioni);
    list cur=S1, vic=NULL;//vicini
    pp padre=NULL;//mi serve per ricordarmi chi era il padre di chi
    int autonomia=0;
    queue q;
    q.head=NULL; 
    q.tail=NULL;
    int maxstazione=0;
    pp temp=NULL; 
    pp pred=NULL;
    bool found=0;
    if(S1==NULL||S2==NULL)
    {
        printf("MI AVEVI DETTO LE STAZIONI C'ERANO!\n");
        return;
    }
    int autonomiaMAX=cerca_autonomiaMAX(S1,S2);
    pred=aggiungi_percorso(S1->el,NULL,NULL);  
    padre=pred; //il padre e il predecessore ora sono uguali al nodo da cui parto ossia S1->el
    vic=cur->succ;
    do
    {
        if((vic!=NULL) && (vic->last>=1))// stavolta prendo quella delle stazioni vicine però. 
            autonomia=vic->Maxheap_car[1];
        else //senno setto autonomia a 0. 
            autonomia=0;
        do 
        {
            if(vic==NULL||vic->el>S2->el||(vic->el)-(cur->el)>autonomiaMAX)
                break; 
            if((autonomia)>=((vic->el)-(cur->el))&&vic->el>maxstazione)
            {
                if(vic==S2)
                {
                    enqueue(&q, vic->el, padre);
                    found=1;
                    break; 
                }
                enqueue(&q, vic->el, padre);
                maxstazione=vic->el;
            }
            vic=vic->succ;
            if((vic!=NULL) && (vic->last>=1))// se c'è almeno una macchina nella stazione prendo la macchina ad autonomia maggiore. 
                autonomia=vic->Maxheap_car[1];
            else //senno setto autonomia a 0. 
                autonomia=0;
        }while(1);
        
        if((q.head)==NULL)//se in qualsiasi momento non mi rimane nella coda "da guardare nessuno nodo ho finito
        {
            printf("nessun percorso\n");
            clean_list(pred);
            //non devo pulire la coda tanto è vuota
            return; 
        }
        if(found==0)
        {
            padre=aggiungi_percorso(q.head->el, q.head->figlio_di, pred); //quando andrò a fare le enque le assegno col valore padre
            pred=padre; //poi ogniqualvolta aggiungo un nodo setto anche il valore predeccesor che mi serve a capire l'ultimo nodo da me inserito.
            cur=cerca_stazione(q.head->el,hash_stazioni);
            vic=cerca_stazione(maxstazione,hash_stazioni);
            vic=vic->succ; 
            //printf("sto per fare la dequeue del valore %d\n",q.head->el);
            dequeue(&q); 
        }
        else
        {
            padre=aggiungi_percorso(q.tail->el, q.tail->figlio_di, pred);
            pred=padre;
            break;
        }
    } while(1);
    temp=pred; //imposto temp all'ultimo nodo che ho messo ossia quello finale.
    while(temp!=NULL)
    {
        printf("%d",temp->el);
        temp=temp->p; 
        if(temp!=NULL)
        {
            printf(" "); 
        }
    }
    printf("\n");
    clean_list(pred);
    clean_queue(q);
    return;
}

sv vista_stazione(sv *stazioni_viste, int key)
{
    //creo il nodo
    sv temp=malloc(sizeof(struct nodo_percorso));
    temp->el=key;
    temp->next=NULL; 
    
    //trovo il posto giusto dove metterlo.
    int hash=key % HLEN;
    sv cur=stazioni_viste[hash];
    sv prec=NULL; 
    while(cur!=NULL)
    {
        prec=cur;
        cur=cur->next;
    }
    if(prec!=NULL)
        prec->next=temp;
    else
        stazioni_viste[hash]=temp;
    return temp;
}

pp aggiungi_percorso(int key, pp figlio_di, pp prev)
{
    pp temp=malloc(sizeof(struct nodo_percorso)); 
    temp->el=key; 
    temp->p=figlio_di; 
    temp->prev=prev; 
    return temp;
}

void clean_queue(queue q)
{
    while(q.head!=NULL)
    {
        dequeue(&q); 
    }
    return; 
}

void clean_hash(sv *stazioni_viste)
{
    sv temp=NULL;
    int i;
    for(i=0;i<HLEN;i++)
    {
        while(stazioni_viste[i]!=NULL)
        {
            temp=stazioni_viste[i]; 
            stazioni_viste[i]=stazioni_viste[i]->next;
            free(temp);
        }
    }
    return;
}

void clean_stazioni(list *stazioni)
{
    list temp=NULL;
    int i;
    for(i=0;i<HLEN;i++)
    {
        while(stazioni[i]!=NULL)
        {
            temp=stazioni[i]; 
            stazioni[i]=stazioni[i]->next;
            free(temp);
        }
    }
    return;
}

void clean_list(pp pred)
{
    pp temp=pred;
    while(pred!=NULL)
    {
        temp=pred; 
        pred=pred->prev;
        free(temp);
    }
    return;
}


void enqueue(queue *q, int el, pp padre)
{
    struct nodo_queue *temp=calloc(1,sizeof(struct nodo_queue)); 
    temp->el=el; 
    temp->figlio_di=padre; 
    if(q->tail!=NULL)
    {
        (q->tail)->next=temp;
        q->tail=temp;
    }
    else
    {
        q->tail=temp; 
        q->head=temp; 
    }
    return; 
}


void dequeue(queue *q)
{
    struct nodo_queue *temp=NULL;
    if(q->head==NULL)
    {
        printf("errore\n");
        return; 
    }
    else
    {
        temp=q->head; 
        q->head=(q->head)->next; 
         if(q->head==NULL)
        {
            q->tail=NULL; 
        }
        free(temp);
        return;
    }
}

list_res inserisci_testa(list_res l, int el)
{
    list_res temp=calloc(1,sizeof(struct nodo_res)); 
    temp->el=el; 
    temp->next=l; 
    return temp; 
}


void printhash(list l)
{
    while(l!=NULL)
    {
        printf("%d->",l->el);
        l=l->next;
    }
    printf("NULL\n"); 
}

void printlist(list l)
{
    while(l!=NULL)
    {
        printf("%d:%d<->",l->el, l->Maxheap_car[1]);
        l=l->succ;
    }
    printf("NULL\n"); 
}

void print_delete(list_res res)
{
    list_res temp;
    while(res!=NULL)
    {
        printf("%d",res->el);
        temp=res;
        res=res->next;
        if(res!=NULL)
        {
            printf(" ");
        }
        else
        {
            printf("\n");
        }
        free(temp); 
    }
}

void printqueue(queue q)
{
    while(q.head!=NULL)
    {
        printf("%d->",q.head->el);
        q.head=(q.head)->next;
    }
    printf("NULL\n"); 
}

bool check(sv *stazioni_viste, int key)
{
    int hash=key%HLEN;
    sv temp=stazioni_viste[hash];
    while(temp!=NULL)
    {
        if(temp->el==key)
        {
            return 1;//l'elemento c'era già 
        }
        temp=temp->next; 
    }
    return 0;
}

int cerca_autonomiaMAX(list S1, list S2)
{
    int max=0; 
    list cur=S1;
    while(cur!=S2)
    {
        if(cur->Maxheap_car[1]>max)
        {
            max=cur->Maxheap_car[1];
        }
        cur=cur->succ;
    }
    if(S2->Maxheap_car[1]>max)
    {
        max=S2->Maxheap_car[1];
    }
    return max; 
}





