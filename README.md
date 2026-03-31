# acid-rain

#include <chrono>
#include <thread>
#include <Windows.h>
#include <vector>


const unsigned FPS = 25;
std::vector<char> frameData;
short cursor = 0;

// Get the intial console buffer.
auto firstBuffer = GetStdHandle(STD_OUTPUT_HANDLE);

// Create an additional buffer for switching.
auto secondBuffer = CreateConsoleScreenBuffer(
    GENERIC_READ | GENERIC_WRITE,
    FILE_SHARE_WRITE | FILE_SHARE_READ,
    nullptr,
    CONSOLE_TEXTMODE_BUFFER,
    nullptr);

// Assign switchable back buffer.
HANDLE backBuffer = secondBuffer;
bool bufferSwitch = true;

// Returns current window size in rows and columns.
COORD getScreenSize()
{
    CONSOLE_SCREEN_BUFFER_INFO bufferInfo;
    GetConsoleScreenBufferInfo(firstBuffer, &bufferInfo);
    const auto newScreenWidth = bufferInfo.srWindow.Right - bufferInfo.srWindow.Left + 1;
    const auto newscreenHeight = bufferInfo.srWindow.Bottom - bufferInfo.srWindow.Top + 1;

    return COORD{ static_cast<short>(newScreenWidth), static_cast<short>(newscreenHeight) };
}

// Switches back buffer as active.
void swapBuffers()
{
    WriteConsole(backBuffer, &frameData.front(), static_cast<short>(frameData.size()), nullptr, nullptr);
    SetConsoleActiveScreenBuffer(backBuffer);
    backBuffer = bufferSwitch ? firstBuffer : secondBuffer;
    bufferSwitch = !bufferSwitch;
    std::this_thread::sleep_for(std::chrono::milliseconds(1000 / FPS));
}

// Draw horizontal line moving from top to bottom.
void drawFrame(COORD screenSize)
{
    for (auto i = 0; i < screenSize.Y; i++)
    {
        for (auto j = 0; j < screenSize.X; j++)
            if (cursor == i)
                frameData[i * screenSize.X + j] = '@';
            else
                frameData[i * screenSize.X + j] = ' ';
    }

    cursor++;
    if (cursor >= screenSize.Y)
        cursor = 0;
}

int main()
{
    const auto screenSize = getScreenSize();
    SetConsoleScreenBufferSize(firstBuffer, screenSize);
    SetConsoleScreenBufferSize(secondBuffer, screenSize);
    frameData.resize(screenSize.X * screenSize.Y);

    // Main rendering loop: 
    // 1. Draw frame to the back buffer.
    // 2. Set back buffer as active.
    while (true)
    {
        drawFrame(screenSize);
        swapBuffers();
    }
}