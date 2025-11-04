## 插件indes.js
```
let myProcessedChatId = null; //全局

eventSource.on('SD_IMAGE_GENERATED', (data) => {
    console.log('收到生成图片事件', data);
    console.log('收到生成图片事件11222', lastProcessedChatId);
    if (ws && ws.readyState === WebSocket.OPEN) {
		console.log('收到生成图片事件111', data);
        ws.send(JSON.stringify({
            type: 'ai_reply',
            chatId: myProcessedChatId,
			      image_txt: data.baseImg,
        }));
    }
});
```

#3 server.js
```
else if (data.type === 'ai_reply' && data.chatId) {
                logWithTimestamp('log', `收到非流式AI回复，发送至Telegram用户 ${data.chatId}`);
                // 确保在发送消息前清理可能存在的流式会话
                if (ongoingStreams.has(data.chatId)) {
                    logWithTimestamp('log', `清理 ChatID ${data.chatId} 的流式会话，因为收到了非流式回复`);
                    ongoingStreams.delete(data.chatId);
                }
                // 发送非流式回复
                if(data.image_txt){
                        const imageBuffer = Buffer.from(data.image_txt, 'base64');
                       // 发送图片到 Telegram
                      bot.sendPhoto(data.chatId, imageBuffer, { caption: '生成完成！' })
                       .catch(err => logWithTimestamp('error', `发送图片失败: ${err.message}`));

                }else {

                  await bot.sendMessage(data.chatId, data.text).catch(err => {
                      logWithTimestamp('error', `发送非流式AI回复失败: ${err.message}`);
                  });
                }
            }
```
## sd 插件/root/SillyTavern/public/scripts/extensions/stable-diffusion
 index.js
 ```
   const filename = `${characterName}_${humanizedDateTime()}`;
    const base64Image = await saveBase64AsFile(result.data, characterName, filename, result.format);
        const imageUrl = `http://155.94.170.142:8000${base64Image}`;
        console.log('Chat imageUrl'+imageUrl);
        console.log('Chat currentChatId'+currentChatId);
        if (eventSource) {
        eventSource.emit('SD_IMAGE_GENERATED', {
                        imageUrl: imageUrl,
                        prompt,
                        generationType,
                        characterName,
                        chatId: currentChatId,
                        baseImg: result.data
        });
    }
    callback
        ? await callback(prompt, base64Image, generationType, additionalNegativePrefix, initiator, prefixedPrompt, result.format)
        : await sendMessage(prompt, base64Image, generationType, additionalNegativePrefix, initiator, prefixedPrompt, result.format);
    return base64Image;
```
 





            
