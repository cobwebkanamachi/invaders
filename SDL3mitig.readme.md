<PRE>
Need SDL3 make it on another dir. Then make this repo.
https://github.com/libsdl-org/
gzip -dc invaders.tar.gz|tar xvf -
cd invaders

if you need GDB debug, please after cmake & make, then bellow.
cmake -DCMAKE_BUILD_TYPE=Debug .

You would need all deps, but manually make this.
cd deps/SDL_sound
make
cd ../..
Then, to make invaders executable do bellow.
bash ./CMakeFiles/invaders.dir/link.txt
/usr/bin/cc -g -rdynamic CMakeFiles/invaders.dir/deps/8080/i8080.c.o CMakeFiles/invaders.dir/deps/SDL_nmix/SDL_nmix.c.o CMakeFiles/invaders.dir/deps/SDL_nmix/SDL_nmix_file.c.o CMakeFiles/invaders.dir/src/main.c.o CMakeFiles/invaders.dir/src/invaders.c.o -o invaders  -L/usr/lib/x86_64-linux-gnu  -L./deps/SDL_sound -lSDL3 -lSDL3_sound

If no sound, please make these and test these.

invaders/audio_test.c: simple sine wave generator for test purpose.
gcc audio_test.c -o audio_test $(pkg-config --cflags --libs sdl3) -lm

find . -name "playsound_simple" -print
./deps/SDL_sound/examples/playsound_simple
find . -name "playsound_simple.c" -print
./deps/SDL_sound/examples/playsound_simple.c
gcc playsound_simple.c -o playsound_simple -I/usr/local/include -L/usr/local/lib -lSDL3 -lm -I../include ../CMakeFiles/SDL3_sound-static.dir/src/SDL_sound.c.o ../CMakeFiles/SDL3_sound-static.dir/src/SDL_sound_wav.c.o

test wav file:./deps/SDL_sound/examples/a.wav (my cheap keyboard play).
Conversations with Geminit how to debug this (esp.no sounds on WSL2).
</pre>
<li><a href="https://github.com/cobwebkanamachi/invaders/blob/SDL3-mitigation/Google%20GeminiQA-J.pdf">Google%20GeminiQA-J.pdf</li><BR>
<li><a href="https://github.com/cobwebkanamachi/invaders/blob/SDL3-mitigation/Google%20GeminiQA-E.pdf">Google%20GeminiQA-E.pdf</li><BR>
<PRE>
This is audio_test.c:
--CUT HERE--
//[cobweb@invaders]$gcc audio_test.c -o audio_test $(pkg-config --cflags --libs sdl3) -lm
//[cobweb@invaders]$./audio_test
#include <SDL3/SDL.h>
#include <math.h>

#define SAMPLE_RATE 44100
#define CHANNELS 2
#define DURATION 3

int main(int argc, char* argv[]) {
if (SDL_Init(SDL_INIT_AUDIO) < 0) {
SDL_Log("SDL_Init Failed: %s", SDL_GetError());
return 1;
}

// 1. オーディオスペックの設定 (Float32)
SDL_AudioSpec spec = { SDL_AUDIO_F32, CHANNELS, SAMPLE_RATE };
SDL_AudioDeviceID dev = SDL_OpenAudioDevice(SDL_AUDIO_DEVICE_DEFAULT_PLAYBACK, &spec);
if (dev == 0) {
SDL_Log("OpenDevice Failed: %s", SDL_GetError());
return 1;
}

// 2. ストリームの作成とバインド
SDL_AudioStream *stream = SDL_CreateAudioStream(&spec, &spec);
SDL_BindAudioStream(dev, stream);

// 3. サイン波データの生成と投入
// 440Hz (ラ) の音を3秒分作成
int num_samples = SAMPLE_RATE * DURATION * CHANNELS;
float *buffer = (float *)SDL_malloc(num_samples * sizeof(float));
for (int i = 0; i < num_samples / CHANNELS; i++) {
float val = sinf(2.0f * M_PI * 440.0f * i / SAMPLE_RATE);
buffer[i * 2] = val; // 左
buffer[i * 2 + 1] = val; // 右
}

// データを一気に投入
SDL_PutAudioStreamData(stream, buffer, num_samples * sizeof(float));

// 4. キューがなくなるまで待機 (ここがポイント)
SDL_ResumeAudioDevice(dev);
SDL_Log("Playing...");

while (SDL_GetAudioStreamQueued(stream) > 0) {
SDL_Delay(100); // 100msごとにキューの残りを確認
}

SDL_Log("Done.");
SDL_free(buffer);
SDL_DestroyAudioStream(stream);
SDL_CloseAudioDevice(dev);
SDL_Quit();
return 0;
}
</PRE>
<BR>Enjoy!
