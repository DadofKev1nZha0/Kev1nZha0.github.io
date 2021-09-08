---
layout: post
title: 通过BlockingCollection实现MessageQueue消息消费 
category: .Net
tag: [.Net]
---

	private readonly BlockingCollection<string> mProcessQueue = new BlockingCollection<string>();
	private void buttonAdd_Click(object sender, EventArgs e)
	{
		for(int n = 0; n < 10; n++)
		{
			string message = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")+ " - " + n; 
            mProcessQueue.Add(message, new CancellationToken(false));
        }
        updatequeue();
    }
    private void buttonTake_Click(object sender, EventArgs e)
    {
        string Action = null;
        mProcessQueue.TryTake(out Action);
        updatequeue();
        while (Action != null)
        {
            richTextBoxTake.Text += Action + "\r\n";
            Application.DoEvents();
            Thread.Sleep(1000);
            mProcessQueue.TryTake(out Action);
            updatequeue();
        }
        //mProcessQueue.CompleteAdding();
        //一旦被Complete之后Queue便无法添加
    }
    private void updatequeue()
    {
        richTextBoxQueue.Text = "";
        if (mProcessQueue.Count > 0)
        {
            foreach (string queued in mProcessQueue)
            {
                richTextBoxQueue.Text += queued + "\r\n";
            }
        }
    }



**参考连接:**

[BlockingCollection 概述](https://docs.microsoft.com/zh-cn/dotnet/standard/collections/thread-safe/blockingcollection-overview)
