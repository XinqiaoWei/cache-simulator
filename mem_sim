/*
 Cache and memory simulator
 */

#include<iostream>
#include<vector>
#include<string>
#include<sstream>
#include<fstream>
#include<math.h>
#include <stdio.h>

using namespace std;

struct CacheStruct{
    int data;
    int byteAddress;
    int LRUCounter;
    int dirtyBit;
};

static int countLRU = 0;


vector<int> createMemory(){
    
    //the max size of memory is 8MB
    int mem_size = 8 * 1024 * 1024;
    
    vector<int> memory (mem_size);
    
    //initialise the memory and set them all to 0
    for(int i=0; i<mem_size; i++){
        memory[i] = 0;
    }
    
    return memory;
}


vector< vector<CacheStruct> > createCache(int addBit, int bt_wd, int wd_bk, int bk_set, int set_ca,
                                            int hitTime, int mem_rt, int mem_wt){
    
    int horizontal = bk_set*(1+1+wd_bk);
    int vertical = set_ca;
    
    vector< vector<CacheStruct> > cache;
    cache.resize(vertical);
    for (int i=0; i<vertical; i++){
        cache[i].resize(horizontal);
    }
    
    for(int i=0; i<vertical; i++){
        for(int j=0; j<horizontal; j++){
            cache[i][j].data = 0;
            cache[i][i].dirtyBit = 0;
            cache[i][j].LRUCounter = 0;
            cache[i][j].byteAddress = 0;
        }
    }
    
    return cache;
}


void Flush(int bt_wd, int wd_bk, int set_index, int bolckStart, vector< vector< CacheStruct> >& cache, vector<int>& memory){
    //Write the previous data to that block and store back to memory before modify
    int data_stored = 0;
    int bolckStart_wt = bolckStart+2;
    int btAddr_store = cache[set_index][bolckStart_wt].byteAddress;
    for(int x=0; x<wd_bk; x++){
        int pre_data = cache[set_index][bolckStart_wt+x].data;
        for(int j=0; j<bt_wd; j++){
            data_stored = pre_data & 0xFF;
            //stores bytes to each memory address
            memory[btAddr_store+j] = data_stored;
            pre_data = pre_data >> 8;
        }
        btAddr_store = btAddr_store + bt_wd;
    }
}

void flush_request(int bt_wd, int wd_bk, int bk_set, int set_ca, vector< vector<CacheStruct> >& cache, vector<int>& memory){
    int dirtyBit_pos = 0;
    int time=0;
    for(int i=0; i<set_ca; i++){
        for(int j=0; j<bk_set; j++){
            dirtyBit_pos = j*(wd_bk+2);
            Flush(bt_wd, wd_bk, i, j, cache, memory);
            cache[i][dirtyBit_pos].dirtyBit = 0;
            time++;
        }
    }
    cout<<"flush-ack "<<time<<endl;
}


void read_request(int address, int addBit, int bt_wd, int wd_bk, int bk_set, int set_ca, int hitTime, int mem_rt, int mem_wt,
                  vector< vector<CacheStruct> >& cache, vector<int>& memory){
    
    int tag;
    int dataRead = 0;
    int time = 0;
    bool tagMatch = false;
    int word_address = address / bt_wd;
    int block_address = word_address / wd_bk;
    int block_index;
    int set_index;
    countLRU++;
    if(bk_set == 1){
        block_index = block_address % (set_ca / bk_set);
        set_index = block_index;
    }else{
        set_index = block_address % set_ca;
    }
    tag = block_address;
    
    if(address >= (8 * 1024 * 1024)){
        cout<<"Invalid address, address is out of range."<<endl;
    }
    
    int cur_tag_pos = 0;
    for(int i=0; i<bk_set; i++){
        cur_tag_pos = i * (wd_bk + 2) + 1;
        //Check tag and valid bit
        if(cache[set_index][cur_tag_pos].data == tag){
            tagMatch = true;
            //Find word to be read
            for(int j=0; j<wd_bk; j++){
                //Specific word matched
                if(cache[set_index][cur_tag_pos+1+j].byteAddress == address){
                    //Then read data
                    dataRead = cache[set_index][cur_tag_pos+1+j].data;
                    cache[set_index][cur_tag_pos-1].LRUCounter = countLRU;
                    time = hitTime;
                    cout<<"read-ack "<<set_index<<" "<<" Hit "<<time<<" "<<dataRead<<endl;
                    break;
                }
            }
            break;
        }else if((cache[set_index][cur_tag_pos].data != tag) && (cache[set_index][i*(wd_bk+2)].data == 1)){	//tag miss
            tagMatch = false;
            continue;
        }
    }
    //Tag miss in cache line, data needs to be written back from memory
    if(tagMatch == false){
        //count value for each block
        int count_offset = wd_bk+1+1;
        int write_index = 0;
        int min = cache[set_index][0].LRUCounter;
        for(int k=0; k<bk_set; k++){
            if(cache[set_index][k*count_offset].LRUCounter < min){
                min = cache[set_index][k*count_offset].LRUCounter;
                write_index = k;
            }
        }
        
        if(cache[set_index][write_index*count_offset].dirtyBit == 1){
            //data in that block back to memory firstly
            Flush(bt_wd, wd_bk, set_index, write_index*count_offset, cache, memory);
            time = hitTime + mem_wt + mem_rt;
        }else{
            time = hitTime + mem_rt;
        }
        
        //read the required data from memory
        int data_from_mem = 0;
        for(int x=0; x<wd_bk; x++){
            for(int k=(bt_wd-1); k>=0; k--){
                data_from_mem = data_from_mem << 8;
                data_from_mem = data_from_mem | memory[address+k];
                
            }
            //Wirte data
            cache[set_index][write_index*count_offset+2+x].data = data_from_mem;
            //Save byte address
            cache[set_index][write_index*count_offset+2+x].byteAddress = address+(bt_wd*x);
        }
        
        dataRead = cache[set_index][write_index*count_offset+2].data;
        //Set Valid bit
        cache[set_index][write_index*count_offset].data = 1;
        //Clear Dirty bit
        cache[set_index][write_index*count_offset].dirtyBit = 0;
        //Set Tag
        cache[set_index][write_index*count_offset+1].data = tag;
        //Set count
        cache[set_index][write_index*count_offset].LRUCounter = countLRU;
        tagMatch = true;
        cout<<"read-ack "<<set_index<<dataRead<<" "<<" miss "<<time<<endl;
    }
    
}

void write_request(int address, int addBit, int bt_wd, int wd_bk, int bk_set, int set_ca, int hitTime, int mem_rt, int mem_wt,
                   int data, vector< vector<CacheStruct> >& cache, vector<int>& memory){
    
    int time = 0;
    int tag = 0;
    bool tagMatch = false;
    countLRU++;
    int word_address = address / bt_wd;
    int block_address = word_address / wd_bk;
    int word_index = word_address % wd_bk;
    int block_index;
    int set_index;
    if(bk_set == 1){
        block_index = block_address % (set_ca / bk_set);
        set_index = block_index;
    }else{
        set_index = block_address % set_ca;
    }
    tag = block_address;
    
    //Check whether data is out of range
    if(data >= pow(16.0, (2*bt_wd))){
        cout<<"Invalid data, data is out of range."<<endl;
    }
    
    
    //if tag matches, and valid bit is 1, then it is a cache hit
    if(bk_set == 1){
        int cur_tag_pos = 1;
        if(cache[set_index][0].data == 1){
            if(cache[set_index][cur_tag_pos].data == tag){
                cache[set_index][cur_tag_pos+1+word_index].data = data;
                cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
                cache[set_index][0].dirtyBit = 1;
                time = hitTime;
                cout<<"write-ack "<<set_index<<" Hit "<<time<<endl;
            }else{
                
                if(cache[set_index][0].dirtyBit == 0){		//Check Dirty bit
                    cache[set_index][cur_tag_pos+1+word_index].data = data;
                    cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
                    cache[set_index][0].dirtyBit = 1;
                    cache[set_index][cur_tag_pos].data = tag;
                    time = hitTime + mem_rt;
                    cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
                }else{
                    Flush(bt_wd, wd_bk, set_index, cur_tag_pos-1, cache, memory);
                    cache[set_index][cur_tag_pos+1+word_index].data = data;
                    cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
                    cache[set_index][cur_tag_pos+1+word_index].dirtyBit = 1;
                    cache[set_index][cur_tag_pos].data = tag;
                    time = hitTime + mem_wt + mem_rt;
                    cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
                }
                
            }
        }else{
            cache[set_index][cur_tag_pos+1+word_index].data = data;
            cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
            cache[set_index][0].dirtyBit = 1;
            cache[set_index][cur_tag_pos].data = tag;
            cache[set_index][cur_tag_pos-1].data = 1;
            time = hitTime + mem_rt;
            cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
        }
    }else{
        int cur_tag_pos = 0;
        for(int i=0; i<bk_set; i++){
            cur_tag_pos = i * (wd_bk + 2) + 1;
            //Check tag
            if((cache[set_index][cur_tag_pos].data == tag) && (cache[set_index][i*(wd_bk+2)].data == 1)){
                tagMatch = true;
                cache[set_index][cur_tag_pos+1+word_index].data = data;
                cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
                cache[set_index][cur_tag_pos-1].LRUCounter = countLRU;
                cache[set_index][cur_tag_pos-1].dirtyBit = 1;
                time = hitTime;
                cout<<"'write-ack' "<<set_index<<" Hit! "<<time<<endl;
                break;
            }
            //tag miss
            else if((cache[set_index][cur_tag_pos].data != tag) && (cache[set_index][i*(wd_bk+2)].data == 1)){
                tagMatch = false;
                continue;
            }
            //valid bit low
            else if(cache[set_index][i*(wd_bk+2)].data == 0){
                cache[set_index][cur_tag_pos+1+word_index].data = data;
                cache[set_index][cur_tag_pos+1+word_index].byteAddress = address;
                cache[set_index][cur_tag_pos-1].dirtyBit = 1;
                cache[set_index][cur_tag_pos].data = tag;
                cache[set_index][cur_tag_pos-1].data = 1;
                cache[set_index][cur_tag_pos-1].LRUCounter = countLRU;
                tagMatch = true;
                time = hitTime + mem_rt;
                cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
                break;
            }
        }
        
        if(tagMatch == false){
            //tag miss, LRU policy is used
            int count_offset = wd_bk+1+1;
            int write_index = 0;
            int min = cache[set_index][0].LRUCounter;
            for(int k=0; k<bk_set; k++){
                if(cache[set_index][k*count_offset].LRUCounter < min){
                    min = cache[set_index][k*count_offset].LRUCounter;
                    write_index = k;
                }
            }
            if(cache[set_index][write_index*count_offset].dirtyBit == 0){
                cache[set_index][write_index*count_offset+2+word_index].data = data;
                cache[set_index][write_index*count_offset+2+word_index].byteAddress = address;
                cache[set_index][write_index*count_offset+1].data = tag;
                cache[set_index][write_index*count_offset].LRUCounter = countLRU;
                cache[set_index][write_index*count_offset].dirtyBit = 1;
                time = hitTime + mem_rt;
                cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
            }else{
                Flush(bt_wd, wd_bk, set_index, write_index*count_offset, cache, memory);
                //Then modifies the block
                cache[set_index][write_index*count_offset+2+word_index].data = data;
                cache[set_index][write_index*count_offset+2+word_index].byteAddress = address;
                cache[set_index][write_index*count_offset+1].data = tag;
                cache[set_index][write_index*count_offset].LRUCounter = countLRU;
                cache[set_index][write_index*count_offset].dirtyBit = 1;
                time = hitTime + mem_wt + mem_rt;
                cout<<"write-ack "<<set_index<<" miss "<<time<<endl;
            }
            
        }
        
    }
}

void debug_req(vector< vector<CacheStruct> >& cache, vector<int>& memory){
    cout<<"Dubug-ack begins!"<<endl;
    
 
    
    cout<<"Debug-ack ends!"<<endl;
}




int main(int argc, char* argv[]){
    
    //Create cache by using 2D-array
    string mem_wt_s = argv[--argc];
    int mem_wt = stoi(mem_wt_s.c_str());
    
    string mem_rt_s = argv[--argc];
    int mem_rt = stoi(mem_rt_s.c_str());
    
    string hitTime_s = argv[--argc];
    int hitTime = stoi(hitTime_s.c_str());
    
    string set_ca_s = argv[--argc];
    int set_ca = stoi(set_ca_s.c_str());
    
    string bk_set_s = argv[--argc];
    int bk_set = stoi(bk_set_s.c_str());
    
    string wd_bk_s = argv[--argc];
    int wd_bk = stoi(wd_bk_s.c_str());
    
    string bt_wd_s = argv[--argc];
    int bt_wd = stoi(bt_wd_s.c_str());
    
    string addBit_s = argv[--argc];
    int addBit = stoi(addBit_s.c_str());
    
    
    vector<int> memory = createMemory();
    vector< vector<CacheStruct> > cache = createCache(addBit, bt_wd, wd_bk, bk_set, set_ca, hitTime, mem_rt, mem_wt);
    
    //Parsing the input
    string cmd;
    while(cin>>cmd){
        if(cmd == "#")
            continue;
        if(cmd == "write-req"){
            string addr, data;
            cin>>addr;
            cin>>data;
            int int_addr = stoi(addr, nullptr, 0);
            int int_data = stoi(data, nullptr, 16);
            write_request(int_addr, addBit, bt_wd, wd_bk, bk_set, set_ca, hitTime, mem_rt, mem_wt,
                          int_data, cache, memory);  
        }
        else if(cmd == "read-req"){
            string addr;
            cin>>addr;
            int int_addr = stoi(addr, nullptr, 0);
            read_request(int_addr, addBit, bt_wd, wd_bk, bk_set, set_ca, hitTime, mem_rt, mem_wt, cache, memory);		
        }
        else if(cmd == "flush-req"){
            flush_request(bt_wd, wd_bk, bk_set, set_ca, cache, memory);
        }
        else if (cmd == "debug-req"){
            debug_req(cache, memory);
        }
    }
    
    
    return 0;
}
